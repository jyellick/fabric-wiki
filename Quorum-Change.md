
#Plan for Quorum Change in OBC (Considerations for the Ledger and PBFT)
 
#### What This Document Does _Not_ Address

This document does not address **three important issues**, which we
defer for later:

1. How will out-to-lunch replicas and clients discover updated
   quorums?  As vukolic@ and jyellick@ have pointed out, a
   replica/client might know an outdated white-list, all of whose
   members are either Byzantine or failed; how will this node learn
   even an _approximation_ to the current quorum?

   Answer: We believe we will need a global membership service, and
   are currently discussing how it will work.  There is known work in
   this area, e.g. [Liskov's work on rBQS][Liskov-rBQS-Automatic-Reconfiguration].

2. How will quorum-change transactions be validated?  signed?  What
   business-process will apply?

   Answer: We believe the separation into _mechanism_ (described
   below) and _policy_ (which quorum-change messages are allowed to be
   committed to the ledger?) will allow us to _later_ implement
   different policies if needed.  But frankly, the standard policy
   will be that, amongst the collection of participating
   organizations, there will be some subset (maybe all) which are the
   "operating governance committee" -- and all of those members will
   need to sign, in order to commit a quorum-change transaction.

3. How will this affect the "window" of PBFT?  (in other words: how
   will this affect PBFT's ability to be pushing concurrent
   transactions/blocks thru consensus?)

   Answer: Currently PBFT _pushes a single block thru consensus at a
   time_ (b/c the ledger requires each block to contain the state-hash
   after application of the prior block).  So the window-size is
   effectively "one".  In the future, once we get MVCC+postimage
   implemented, we can have nontrivial windows -- at that point:

     * the window-size will be part of PBFT's durable configuration  
	 * A quorum-change transaction committed at block-height H, with
       window-size W, must become effective later than block-height
       _H+W+1_.  
	 * PBFT, when it learns that a quorum-change is pending, will
       "collapse" its window, allowing **no proposed transactions** at
       or past the effective block-height of the quorum-change.  
     * Once the view-change has installed the new quorum, of course
       there will be no pending quorum-change (or maybe there'll be
       yet another, in the further future), and the window will return
       to its configured size.

## A High Level Plan

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
    accordingly.  How does a replica get the updated white list if the
    network has grown? Conceivably all previously white-listed peers
    could now be byzantine.  I'm sure this is thought out, but should
    be described explicitly.

3. At the prescribed commit-number from (2) above, PBFT will "force a
   view-change" to cause the new configuration to go into
effect.  
  * hence, a view-change must re-read all durable configuration from
    the state.

  * [jyellick@] An alternate proposal is instead of performing the
    view change at that commit, instead, simply add the new white-list
    member to a sezt of 'super view changers', giving the new member a
    one time use 'everyone perform a view change to let me join the
    network' request.  In this way, we could leverage the existing
    view change logic for picking a checkpoint for the new VP to begin
    executing from.  Once the new VP joins the network, this view
    change power would be revoked.

  * [chetmurthy@] (First of two comments): I'm not sure what you're
    proposing here; part of the idea of using a commit to "install" a
    new view (effective at a commit-number in the future) is to make
    things compatible with 'windows' (more than one decree in-process
    to be committed concurrently).  So the quorum-change commit's
    "effective-as-of" commit-number must be "further out" than the
    window-size; so that all participants, once they see this
    quorum-change commit, will know to "collapse" the window,
    forbidding new trans beyond the quorum-change commit until after
    the quorum-change actually takes effect.

  * [chetmurthy@]But another design rule I think we should follow
    (best enunciated by Mark Hayden) is:

	>If it's something that happens rarely, don't try to over-think
	 it: do it in the most straightforward way possible.

    I'd like to claim that that's what we're aiming for here: we'll
    use -only- referenced to committed updates to drive state-changes,
    in a manner that should not cause errors, regardless of where
    failures occur

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

    * [jyellick@] Today, there is theoretically a 'snapshot' at every
      block.  When PBFT issues a checkpoint message, all the VPs agree
      on a particular block hash for that checkpoint.  This is how the
      current state transfer code is informed of 'snapshots' which can
      be transferred to.

    * [chetmurthy@] We have a snapshot (and a hash), but a client
      connecting to a peer to download the state, doesn't know that
      the peer is trustworthy (as you asked, right?)  To know that, we
      need to ask _3f+1_ peers for their hashes at that block-height,
      right?  And that mechanism doesn't exist (yet).

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
and even if they did, they only have the blockchain, not the state.

  * [jyellick@] Half of this is most likely a relatively simple fix, `noops`
	invokes a method `notifyBlockAdded` after a block has been
	committed, which creates the relevant message and broadcasts it to
	the NVPs.  The reluctance to add this to PBFT is because this
	seems like an odd thing for consensus to be doing, that instead
	this should probably be handled somewhere deeper in the peer, as
	this should be a common feature of all consensus implementations.

  * [jyellick@] The other half of this is the fact that NVPs do not perform state
	transfer today.  If they are live on the network to receive
	broadcast blocks then their blockchain stays up to date, but I
	believe they do not fill in any gaps in their blockchain.  There
	are a couple issues already out there around this #645 and #640,
	but a new issue to actually implement this for NVPs should also be
	created.  Still, this should be the same common state transfer
	code used by consensus, so hopefully this is not a huge piece of
	work.

  * [chetmurthy@] My belief is we should **not** try to solve this
    problem right now, but rather wait until we have consensus
    sufficiently cleaned-up so that we can do it "right" -- perhaps
    even wait until we have the new pub/sub implementation working.

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

  * [jyellick@] The above seems like we're back to the same chicken or
    the egg problem.  If we know which _3f+1_ to poll, then we already
    have our white list, and the lack of white list is the whole
    reason we're going through this multi-step process.  Why not just
    directly connect at this point?  (There are potentially some
    missing configuration parameters, but these could be picked up in
    much the same way that a VP which was shut down at sequence number
    0 and is reconnecting at sequence number 100k)

    There's a second problem potentially.  Asking "what is the current
	block-height" is a less simple question than it seems.  It's very
	likely that each peer is at a different block height.  This is why
	the existing `HELLO` mechanism is insufficient for PBFT.  The only
	promise given by PBFT regarding block height is that when a
	checkpoint message is issued, the VP is at least at the block
	height corresponding to that checkpoint.  I think instead the
	correct question is "What checkpoints are you aware of" and then
	looking for the highest numbered checkpoint with f+1 matching
	replies.  This is very much the logic of a view change operation.
	Note, that it's still possible that in a fast network the
	watermarks may move before the new VP joins, or in a very fast
	network, the watermarks could move while the strong read is still
	occurring, and there might not be f+1 intersecting checkpoints.
	This is a problem the view change power for a new VP might solve.

  * [chetmurthy@] Two comments:  
	1. Indeed, discovering the current quorum-membership goes back to
       the quorum-membership-service point above.  But this is
       different from the situation for active peers, b/c those active
       peers _already know_ the current quorum-membership: they are
       part of the change process.  Here, we're talking about a node
       which is effectively a _client_, so it needs to do what clients
       do when they have minimal trust: "ask _3f+1_ members", right?  
    2. [Jason, can you please comment most especially on this?] When I
       ask "what is the current block-height", I'm really asking a
       question of the _ledger_, not of PBFT.  **PBFT should not come
       in to it at all**.  All the node needs, is a sufficiently
       complete committed log and state, and it should be able to
       switch over and join.

		Yes? [Jason, please comment?]

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
