
# Bitcoin Does Not Solve Byzantine Generals

Recently [Emin Gun Sirer][Sirer2016] argued that Bitcoin provides
strong consistency, not eventual consistency (as was argued by
[Bitcoin Meets Strong Consistency][DeckerSeidelWattenhofer2014]).  In
this short note, I want to do three things:  
* **provide a brief counter-example**.  It suffices to understand only
  the publicly known properties of Bitcoin, to prove that Bitcoin
  cannot offer antyhing that anybody would _recognize_ as "strong consistency".  
* **recap Sirer's argument** in detail, pointing out where I believe his
  mistake lies.  
* **Argue that _consensus_[Wikipedia-Consensus] and not _strong
consistency_ is the right property**.  Consensus is the right property
to ask of a distributed database suitable for building WAN-scale
financial systems, and Bitcoin does not provide it.

## A Brief Counter-example

Sirer's argument is that Bitcoin provides "strong consistency".  By
this, he means that, even though at any point, 
>the last few blocks of a blockchain are subject to rearrangement  
if one waits until a few blocks more have been appended to the
blockchain, then the _rest_ of the blockchain is _not_ subject to
rearrangement in any real-world sense.  Hence, if we look back six
blocks in the chain, the blockchain's answers to questions like "did
Joe make that payment of $50k for my Maserati" are not subject to
revision.

This is false for one simple reason:

    Network can and do partition, and for unpredictably long times.


[Sirer2016]: http://hackingdistributed.com/2016/03/01/bitcoin-guarantees-strong-not-eventual-consistency/
[DeckerSeidelWattenhofer2014]: http://arxiv.org/pdf/1412.7935.pdf
[Wikipedia-Consensus]: https://en.wikipedia.org/wiki/Consensus_(computer_science)
