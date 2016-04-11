
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

