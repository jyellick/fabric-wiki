
# Bitcoin Does Not Solve Byzantine Generals

Recently [Emin Gun Sirer][Sirer2016] argued that Bitcoin provides
strong consistency, not eventual consistency (as was argued by
[Bitcoin Meets Strong Consistency][DeckerSeidelWattenhofer2014]).  In
this short note, I want to do three things:  
* provide a brief counter-example, based only on publicly known
  properties of Bitcoin, that it _cannot_ provide anything that
  anybody would _recognize_ as "strong consistency".  
* recap Sirer's argument in detail, pointing out where I believe his
  mistake lies.  
* Argue that in fact, what we ought to be asking ourselves, is whether
  Bitcoin provides [_consensus_][Wikipedia-Consensus] (in the classic
  formulation due to Lamport, Fischer, Lynch, Patterson, and others).
  And what we'll find, is that Bitcoin does _not_ provide consensus.



[Sirer2016]: http://hackingdistributed.com/2016/03/01/bitcoin-guarantees-strong-not-eventual-consistency/
[DeckerSeidelWattenhofer2014]: http://arxiv.org/pdf/1412.7935.pdf
[Wikipedia-Consensus]: https://en.wikipedia.org/wiki/Consensus_(computer_science)
