
# Bitcoin Does Not Provide Strong Consistency

Recently [Emin Gun Sirer][Sirer2016] argued that Bitcoin provides
strong consistency, not eventual consistency (as was argued by
[Bitcoin Meets Strong Consistency][DeckerSeidelWattenhofer2014]).  In
this short note, I want to do two things:  
* **provide a brief counter-example**.  It suffices to understand only
  the publicly known properties of Bitcoin, to prove that Bitcoin
  cannot offer anything that anybody would _recognize_ as "strong consistency".  
* **Briefly re-explain [_consensus_][Wikipedia-Consensus] ** and
  re-cap the counter-example, as a proof that Bitcoin cannot solve the
  consensus problem.

First though, why should any of this matter?  Bitcoin seems like a
simple payment model, requiring a trivial database (one four-column
table).  If, from time-to-time, an inconsistent set of transactions
turns up, due to a too-long partition, perhaps that's the price we pay
for a global-scale payment system.  The problem with this attitude, is
that _payment systems in the real world_ aren't trivial.  The
[history of payment systems][Nacamuli-Payment-Systems] tells us that
modern payment systems are complex for reasons having _nothing_ to do
with the underlying technology.  They must deal with (at a minimum):  
* **default risk**: (banks keep track of how much they're owed by other
  banks, and this forces those other banks to reorder their payments
  to minimize their balances)  
* **liquidity costs**: banks may need to borrow funds short-term to make
  payments (the basic function of banks, let's remember, is to borrow
  short and lend long)  
These issues directly correspond to complexity in the database
application.  Similarly, most retail brokerage applications (another
touted application of blockchains) have a complex data-models --
including details like tax-lot calculations, multiple and complex
access-control, etc, etc, etc.

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
[the network is reliable.][Aphyr], so that never happens.

## Consensus and why Bitcoin doesn't achieve it

Consensus is [defined by Lamport][Lamport-Paxos-Made-Simple] in terms
of three properties of a system that _chooses_ values:  
* **Integrity** Only a value that has been proposed, may be chosen.  
* **Agreement** Only a single value is chosen.  
* **Validity** A process never learns that a value has been chosen,
unless it actually has been.

In the context of a blockchain, we can think of consensus in terms of
the blocks: we achieve consensus on the value of each block,
one-by-one.  Each process participating in the system (each _miner_)
is choosing blocks, and to say that the protocol the miners execute,
achieves consensus, means precisely that the values chosen by the
miners satisfy the three properties above.  

    And in this presentation, we can assume that the meaning "choose
    the value of block N" is "choose the value of block N, afer
    Omega=6 blocks have buried block N"

It should be clear now that Bitcoin (in the face of arbitrary network
partitions) cannot achieve **Agreement**: on each side of the
partition, miners choose the value of a block, and if the partition
lasts long enough, that choice outlasts Omega .... so applications
that rely on this definition of "choose" _will_ act on the assumption
that the block is irrevocable, only to find (on one side of the
partition) that when the partition heals, the block is indeed revoked.

[Sirer2016]: http://hackingdistributed.com/2016/03/01/bitcoin-guarantees-strong-not-eventual-consistency/
[DeckerSeidelWattenhofer2014]: http://arxiv.org/pdf/1412.7935.pdf
[Wikipedia-Consensus]: https://en.wikipedia.org/wiki/Consensus_(computer_science)
[Aphyr]: https://aphyr.com/posts/288-the-network-is-reliable
[Lamport-Paxos-Made-Simple]: http://research.microsoft.com/en-us/um/people/lamport/pubs/paxos-simple.pdf
[Nacamuli-Payment-Systems]: http://www.amazon.com/Payment-Systems-Macmillan-Financial-Institutions/dp/0230202500
