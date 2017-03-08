---
layout: post
title: The Smart Contract and Transactional Memory Analogy
---

In this post and the coming ones I will be detailing about a particularly interesting analogy between Smart Contracts and Transactional memory. And how we can use one to verify and reason about the other. The majority of this work has been driven by a [paper by Dr. Ilya Sergey and Dr. Aquinas Hobor](http://ilyasergey.net/papers/csc-wtsc17.pdf) on similar grounds. My attempt has been to try and extend the ideas from this paper, to mitigate the issues plaguing Smart contracts and their semantics as listed in another paper on [Making Smart Contracts Smarter](https://eprint.iacr.org/2016/633.pdf). Essentially this serves as a literature survey as well as a playground for me to pitch my ideas unifying the 2 papers. Sounds rad? Then read ahead. :)

**THE POWER OF ANALOGY**

Before delving into all the gory details, I will like to take some time out to talk a little about the power of analogies in Science. 
Science itself is littered with example of analogies throughout its history. I would like to point you to a very well known example in Computer Science. The Turing Award winner Leslie Lamport, arguably the greatest computer scientist of the current generation, has spent his entire life studying distributed systems. His seminal work on consensus algorithm known as the [Paxos algorithm](http://lamport.azurewebsites.net/pubs/lamport-paxos.pdf) is based on an analogy with a "Part-time parliament" in the Greek town of Paxos. The parliament functioned despite of the peripatetic propensity of its part time legislators. Quoting from the paper:
```
The problem of governing with a part-time parliament bears a remarkable correspondence
to the problem faced by today’s fault-tolerant distributed systems, where
legislators correspond to processes and leaving the Chamber corresponds to failing.
The Paxons’ solution may therefore be of some interest to computer scientists.
``` 
Analogies are easy to remember and if fully understood, can inspire new research ideas. However the negative consequences of an analogy is that people can misunderstand and and start applying it too broadly to the corresponding field. The key to understand is that an analogy is never a complete argument but rather an introductory remark to start a more complete discussion analyzing the merits and limits of one field in another. Keeping that in mind let us start by understanding what are smart contracts and the issues surrounding them.

**SMART CONTRACTS AND THEIR ASSOCIATED PERILS**

