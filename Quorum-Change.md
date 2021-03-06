
#Plan for Quorum Change in OBC (Considerations for the Ledger and PBFT)

This document is a description of the plan for implementing quorum
change in OBC's PBFT.  The basic plan is to use the already-existing
state-transfer and view-change mechanism of PBFT.  A more detailed
plan is below, followed by sections explaining particular points more
fully.

1. Ensure that all durable state of OBC and PBFT is persisted in the
'state'.  Specifically:  
  * chaincode deployment currently does not store the chaincode source
    and other relevant parameters in the state
    [Issue 1054](https://github.com/hyperledger/fabric/issues/1054)  
  * PBFT itself has a few parameters (quorum whitelist, "f" parameter,
    etc) that are not persisted in the state.  Call this the "_durable
    configuration_".  
  * The PBFT durable configuration should be stored in a single
    'system' chaincode k/v store, and preferably as a single protobuf
    -- we need a design for this DB.  
    * this PBFT durable configuration needs to be versioned (to be
      described below)

2. Build a mechanism (chaincode) to update that PBFT durable
configuration, such that a new configuration will take effect as of a
commit number (blockchain height) in the future, and when that update
is installed, PBFT is informed of this commit-number.  
  * during PBFT startup, PBFT will consult the state to determine its
    durable configuration, and will fetch the configuration based on
    the current commit-number.  
  * of course (during startup) PBFT might also find a new
    configuration to take effect in the future, and should act
    accordingly

3. At the prescribed commit-number from (2) above, PBFT will "force a
   view-change" to cause the new configuration to go into
effect.  
  * hence, a view-change must re-read all durable configuration from
    the state.

4. Build a mechanism to allow a new peer to act as a client, connect
to the current quorum, transfer the current blockchain and
state-snapshot.  
  * jyellick@ notes that these data need to be trustworthy, and we
    _also_ need a mechanism to verify that.  But we can defer this
    until later  
    * We can ensure trustworthiness of the blockchain using "strong
      reads"  
    * state-transfer needs to eventually use the same mechanism, but
      it won't be feasible until all peers take a state-snapshot
      simultaneously.

## PBFT Changes and Durable Configuration

We'll store the PBFT durable configuration in a chaincode state.  The
keys will be something like "pbft-config:0000NNNN" (zero-padded so it
sorts right) with NNNN being the commit-number at which the
configuration takes effect.  The value will be a serialized proto (I'd
prefer text-serialized (for debuggability), but am not picky) and
should contain _all_ durable config of PBFT.  A system chaincode will
install the new configuration, (eventually) checking that the
transaction was properly signed, and that the key is suitably in the
future.  It might as well also provide access to the current
configuration, and (eventually) will garbage-collect out-of-date
configurations.

At PBFT startup (or view-change), it will need to be given two things:  
* current (on-disk) blockchain block-height  
* access to the state  
With these two data, PBFT can fetch the current configuration, and
discover whether there is a pending "next configuration".

The state-update process will also need to be modified so that system
chaincodes are thus-marked (read-below).

### Managing the "system catalog chaincode store"

We need to decide if all system parameters should go in a _single
chaincode k/v store_ or in a number of chaincodes.  I would go with
the former for simplicity.  Call this k/v store the "system catalog
chaincode".  PBFT needs to be informed when the PBFT-relevant part of
this system catalog changes; it is possible that other code will need
to be informed about other changes.  Storing all system catalog state
in a single chaincode, would allow a single check during state-delta
application, to decide whether a detailed scan should be done to
update relevant subsystems.  
* For instance, the code that updates PBFT needs to keep track of the 
last update it performed, so that PBFT will only get told once per update,
and so that it will get told for certain during startup, if there's a pending
update.

## Spinning Up a New Peer

There are three problems in spinning up a new peer:  
1. currently, non-validating peers don't really work (per jyellick@)
and even if they did, they only have the blockchain, not the state  
2. when the peer is retrieving the blockchain and state from the
current quorum-set, it has no way of validating that that
blockchain/state is correct (an issue of trust).  
3. once a "new peer" has a moderately up-to-date blockchain and state,
it needs to actually "switch" into being a peer.  We'll address these
in order 2, 1, 3. 

### Trust and verifying downloaded blockchain and state

Today, a client downloading a blockchain from a validating peer has no
way of ascertaining that that peer isn't lying.  ***In the future**
(not for today), a client will be able to perform a "strong read" to
ascertain this: >a **strong read** would consist in running a
transaction that is read-only, and subscribing to _3f+1_ peers to
observe the results of that transaction.

In the case of the blockchain itself, the strong read would be asking
the question 
>What is the current block-height, and what is the block-hash at that height? 
With the answer to this question from _2f+1_ peers (amongst them, the
peer from which it's getting its blockchain), the client can proceed
with verification of the integrity of its downloaded blockchain.

In a similar manner, today the blockchain records state-hashes, so the
client can verify that the state it's transferred (which is at a
particular block-height) has the right hash.

**Note Well**: When we move the blockchain out of rocksdb, and remove
  merkle-tree state-hashing from the blockchain, we'll need to
  periodically take persistent state-snapshots and hash them,
  simultaneously on all peers.  This isn't complicated, and should be
  much more efficient than the current mechanism (since such snapshots
  will occur very infrequently).

### Non-Validating Peers: Clients downloading blockchain and state

**Eventually** if OBC's PBFT/Sieve doesn't currently have full support
for non-validating peers, we need to extend it to do so.  For now, it
suffices for a client-that-wants-to-be-a-peer to do the following:  
* perform a full state-transfer  
* **then** download the entire blockchain, until the block for which it
received the full state-transfer.  
At this point, it will have a full and consistent
blockchain-plus-state.  As long as the client is doing this _after_
the quorum-change in which the client was added to the
quorum-whitelist, the client will get a state containing that
quorum-whitelist, and this will be ready to join the quorum.

### Actually joining the quorum

Once a client has downloaded the blockchain and state,
properly-formatted on-disk, it can switch to being a peer, by
restarting, and there is no good reason for this to be other than a
really, really, really heavyweight operation.  All the work should be
in properly arranging files on-disk, so that the switchover is
trivial.  Basically, the client should be able to fire up the peer,
pointed at the newly-downloaded blockchain/state, and all should
proceed as if the peer was existing since the genesis block.
