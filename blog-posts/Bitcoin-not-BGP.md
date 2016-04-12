
# Bitcoin Does Not Solve Byzantine Generals

Recently [Emin Gun Sirer][Sirer2016] argued that Bitcoin provides
strong consistency, not eventual consistency (as was argued by
[Bitcoin Meets Strong Consistency][DeckerSeidelWattenhofer2014]).  In
this short note, I want to do three things:  
* **provide a brief counter-example**.  It suffices to understand only
  the publicly known properties of Bitcoin, to prove that Bitcoin
  cannot offer anything that anybody would _recognize_ as "strong consistency".  
* **recap Sirer's argument** in detail, pointing out where I believe his
  mistake lies.  
* **Argue that [_consensus_][Wikipedia-Consensus] and not _strong
consistency_ is the right property**.  Consensus is the right property
to ask of a distributed database suitable for building WAN-scale
financial systems, and Bitcoin does not provide it.

## A Brief Counter-example

Sirer's argument is that Bitcoin provides "strong consistency".  By
this, he means that, even though at any point,  

>the last few blocks of a blockchain are subject to rearrangement  

there is some number of blocks Omega such that, if we discard those
last Omega, then the _rest_ of the chain is unchanging and
consistent.  From this, arises a recipe for "knowing if you got paid":  
1. look for your transaction in the blockchain  
2. wait until Omega blocks have appeared "above" your tran (in the
   parlance, your transaction has been "buried" by Omega blocks)  
3. look again for your transaction; if it still appears in that block,
   you can trust that you got paid.  
Since Bitcoin generates blocks more-or-less every ten minutes, this
corresponds directly to a waiting period.

So let's make this crystal-clear.  The argument is:

    If we can put an upper bound on how long we need to wait, for the
    block in which our tran is found to stabilize, then effectively
    Bitcoin (with that wait) is a consistent system.

This is false for one simple reason:

    Networks can and do partition, and for unpredictably long times.

If the bitcoin network partitions, then each partition (side of the
network) can and will proceed onwards, mining new blocks and extending
the blockchain.  This is touted as a _strength_ of Bitcoin (and is the
essence of "permissionlessness": in a sense, each partition acts as if
the other side has just "exited" the network).  When the network
partition is repaired, one or the other side will find that _all_ the
blocks mined since the partition have disappeared.

I want to make another thing crystal-clear:

    Sirer is right, that if you wait Omega = 6 blocks, the chance of
    seeing an inconsistency is vanishingly small.

Until a partition that lasts longer than an hour.  And as we all know,
[the network is reliable.][Aphr].


[Sirer2016]:
http://hackingdistributed.com/2016/03/01/bitcoin-guarantees-strong-not-eventual-consistency/
[DeckerSeidelWattenhofer2014]: http://arxiv.org/pdf/1412.7935.pdf
[Wikipedia-Consensus]: https://en.wikipedia.org/wiki/Consensus_(computer_science)
[Aphyr]: https://aphyr.com/posts/288-the-network-is-reliable
