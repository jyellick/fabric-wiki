
#Fisking "Why Bitcoin Matters"

In early 2014
[Marc Andreesen wrote a blog post][Andreesen-Why-Bitcoin-Matters]
arguing that Bitcoin is a pivotal technology for financial
transactions.  He makes a number of expansive claims in the post,
which I feel are categorically false, and need to be debunked.  Hence,
this post.

In this post, I'll focus on two issues:  

* Bitcoin is not a solution to any distributed consensus problem
  worthy of the name

* Bitcoin is _wildly_ more fraud- and crime-friendly than the current
  monetary system, and for good reason

## Bitcoin does not "solve Consensus"

>First, Bitcoin at its most fundamental level is a breakthrough in
computer science ... Bitcoin is the first practical solution to a
longstanding problem in computer science called the Byzantine Generals
Problem.  More generally, the B.G.P. poses the question of how to
establish trust between otherwise unrelated parties over an untrusted
network like the Internet.

Bitcoin is **most assuredly not** a solution to BGP in the real world.
Academics have written papers about Bitcoin and **Synchronous BGP**
(wherein, one assumes that the network does delay messages infinitely
-- thus, networks cannot partition).  But I have found no papers about
Bitcoin and **Asynchronous BGP**; it doesn't take a lot of reflection
to conclude that if a network partitions, the bitcoin miners on _both
sides_ will continue to operate (and hence, will diverge in their idea
of the blockchain's contents).  In the language of the BGP, if the
messengers between the generals never get thru, _obviously_ the
generals will never be able to agree.

>The Bitcoin ledger is a new kind of payment system. Anyone in the
>world can pay anyone else ... and in many cases, no fees .... Bitcoin
>is the first Internetwide payment system where transactions either
>happen with no fees or very low fees (down to fractions of pennies).

This again is false, because there _is_ a fee: the bounty to a miner
for mining a block.  In economics, this is referred to as seigniorage.
On April 12 2016, one BTC is worth $428, and the average block
contains between 1000 and 1500 transactions, yielding a seigniorage of
between 28 and 42 cents per transaction.

## Bitcoin is a haven for crime and ransomware

The worst howler in the piece concerns fraud and crime, and it goes to
the heart of why Bitcoin is a terrible idea:

>Finally, I’d like to address the claim made by some critics that
>Bitcoin is a haven for bad behavior, for criminals and terrorists to
>transfer money anonymously with impunity. This is a myth, fostered
>mostly by sensationalistic press coverage and an incomplete
>understanding of the technology. Much like email, which is quite
>traceable, Bitcoin is pseudonymous, not anonymous. Further, every
>transaction in the Bitcoin network is tracked and logged forever in
>the Bitcoin blockchain, or permanent record, available for all to
>see. As a result, Bitcoin is considerably easier for law enforcement
>to trace than cash, gold or diamonds.

Even at the time, in 2014, there was widely-deployed ransomware
demanding payment in Bitcoin, per
([Varonis][Varonis-Brief-History-of-Ransomware]), and a vanishing
number of ransomware demanding payment via _any other payment
mechanism_.  The claim that, because Bitcoin's ledger is publicly
visible, criminal activity can be tracked-down, was manifestly false
then, and time has shown us just how false it is.  Numerous
([reports][Cryptowall-Report]) have shown us that Bitcoin is a
thieves' paradise.

What's even more amazing, is that the above words were written a
_half-year after the Mt. Gox_ collapse.  If Bitcoin had _any_ of the
purported properties, one would think that they would have resulted in
the swift (or at least, near-complete) recovery of all stolen coins.
Instead, in the Mt. Gox collapse 200,000 of 850,000 BTC were lost.  By
contrast, in the [MF Global collapse]:[MF-Global-collapse] 93 percent
of all funds were recovered.  In the recent case of the
[Bangladesh Central Bank theft][Bangladesh-bank-theft], $21 million
out of $101 million has so far been recovered, and it is instructive
to reflect on _why_ that recovery has been possible:

    The financial system today is _strongly_ biased (not enough, as it
    turns out) towards "know your customer", so tracing illicit
    transfers of funds is at least possible.

The problem is when those funds are sent into jurisdictions with lax
laws and supervisory practices, like the Philippines.  Or Bitcoin.



[Andreesen-Why-Bitcoin-Matters]: https://a16z.com/2014/01/21/why-bitcoin-matters-2/

[Varonis-Brief-History-of-Ransomware]: https://blog.varonis.com/a-brief-history-of-ransomware/

[Cryptowall-Report]: http://cyberthreatalliance.org/cryptowall-report.pdf

[MT-Gox-collapse]: https://en.wikipedia.org/wiki/Mt._Gox#Insolvency_and_shutdown

[MF-Global-collapse]: https://en.wikipedia.org/wiki/MF_Global

[Bangladesh-bank-theft]: http://www.bloomberg.com/news/articles/2016-03-09/the-1-billion-plot-to-rob-fed-accounts-leads-to-manila-casinos
