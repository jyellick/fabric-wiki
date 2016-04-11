
#Plan for Quorum Change in OBC (Considerations for the Ledger and PBFT)

This document is a description of the plan for implementing quorum
change in OBC's PBFT.  The basic plan is to use the already-existing
state-transfer and view-change mechanism of PBFT.  A more detailed
plan is as follows:

1. Ensure that all durable state of OBC and PBFT is persisted in the
'state'.  Specifically:

  * chaincode deployment currently does not store the chaincode source
    and other relevant parameters in the state

  * PBFT itself has a few parameters (quorum whitelist, "f" parameter,
    etc) that are not persisted in the state

  * All PBFT state should be stored in a single 'system' chaincode k/v
    store, and preferably as a single protobuf -- we need a design for
    this DB.

2. Build a mechanism to allow a new peer to act as a client, connect
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

