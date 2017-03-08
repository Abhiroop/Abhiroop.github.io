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
Analogies are easy to remember and if fully understood, can inspire new research ideas. However the negative consequence of an analogy is that people can misunderstand and and start applying it too broadly to the corresponding field. The key to understand is that an analogy is never a complete argument but rather an introductory remark to start a more complete discussion analyzing the merits and limits of one field in another. Keeping that in mind let us start by understanding what are smart contracts and the issues surrounding them.

**SMART CONTRACTS AND THEIR ASSOCIATED PERILS**

A smart contract is a program that runs in the blockchain and has its correct execution enforced by the consensus protocol. A smart contract can be thought analogous to an object in OOP terminology. It encapsulates state within itself. This state is manipulated by a set of functions which are called either directly by the clients or indirectly by another smart contracts. Smart contracts are defined using Turing complete languages like [Solidity](https://solidity.readthedocs.io/en/develop/). In this post we will be using Solidity, a static typed language, for all of our examples which will be executing on top of the [Ethereum blockchain](https://www.ethereum.org/). 

The execution semantics of a transaction[1] in the Ethereum block chain should ideally exhibit *atomicity* and *consistency*. The paper by Lu, et al find at fault this very assumption. They pinpoint 4 primary problems existent in the semantics of Smat Contracts. They are:
* Transaction-Ordering Dependence
* Timestamp Dependence
* Mishandled Exceptions
* Reentrance Vulnerabilities

We will briefly define each of these problem and later see how by introducing **lazy transactional semantics** to Solidity, we can atleast solve 2 of them. Read on.

Have a look at this piece of code:

```javascript
contract MarketPlace{ 
    uint public price; 
    uint public stock; 
    /.../
    function updatePrice(uint _price){ 
        if (msg.sender == owner)
            price = _price;
    function buy (uint quant) returns (uint){
        if (msg.value < quant * price || quant > stock) 
            throw;
        stock -= quant;
        /.../ 
    }}
```
The syntax is very similar to vanilla javascript (Never thought I would have to mention javascript in my Haskell blogs :sweat: )


[1]A transaction is something which results in the execution of a smart contract.