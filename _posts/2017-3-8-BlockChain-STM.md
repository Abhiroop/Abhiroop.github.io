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

* **Transaction Ordering dependence**

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
Barring the types, the syntax is very similar to vanilla javascript (Never thought I would have to mention javascript in my Haskell blogs :( ). The point of interest are the `buy` and `updatePrice` function. 

Now the names of the functions are pretty descriptive themselves. So forget smart contracts for a moment and lets think in real life. You want to `buy` a product from Amazon at 10$. You decide to go ahead and start paying for the goods. However while you are mentioning your card details, the seller decides to call an `updatePrice` function on the product and change the price to 15$. Now when you actually pay through the payment portal you end up transferring only 10$ because that is the price that you saw, but meanwhile the amount you paid is 5$ short of the amount the product is listed for. So what happens to the state of the transaction? It lies in an insconsistent state. This fact can be used by malicious users to their benefits as defined in the paper bu Lu, et al.  Buyers may have to pay much higher than the observed price when they issue the buy requests. ANd being a decentralized system there is no uniform notion of time in the blockchain. We will talk a lot more about this vulnerability later, and how mitigitaing this is much more difficult than the other vulnerbailities. However, in the meanwhile, let us detail the other vulnerabilities.

* **Timestamp Dependence**

This is an age old problem. Researchers have spawned a lot of papers citing the problems caused by using timestamp for pseudo-random number generation. These problems become thousandfold when the timestamp is issued in a distributed system and that too by the participants themselves. Moreover Ethereum allows the miner to vary the value by roughly 900 seconds. The paper by Lu, et al demonstrates a similar example on a contract called `theRun`. Allowing contracts to access to the block timestamp is essentially a redundant feature that renders contracts vulnerable to manipulation by adversaries.

* **Mishandled Exceptions**

In etehereum there are multiple ways for a contract to call another. One of them is calling the other contracts via the `send` instruction. If there is an exception raised in the callee contract, it terminates, reverts its state and returns False. Howvere there are cases when the exceptions tends to not get propagated to the caller contract. Research has shown, 27.9% of the contracts do not check the return values after calling other contracts via send. This can render the caller contract to an inconsistent state and we study that in the KoET(KingOfTheEtherThrone) example below.

```javascript
1 function claimThrone(string name) {
2     /.../
3     if (currentMonarch.ethAddr != wizardAddress) 
4         currentMonarch.ethAddr.send(compensation); 
5     /.../
6     // assign the new king
7     currentMonarch = Monarch(
8         msg.sender, name,
9         valuePaid , block.timestamp);
10 }}
```
The KoET contract is very simple. The person bidding to be the King pays the current King an amount in ethers. The current king profits from the difference of the price he paid to the king before him and the price others pay to be his successor. Now if we look at line 4 and line 7 above the vulnerability becomes apparent. We are executing line 7 and dethroning the current king without checking if line 4 threw an exception. And once we crown the new king the behavior becomes irreversible. The exact reason for the exception varies from case to case but the basic gist is that the contracts do not exhibit proper defensive programming strategies while dealing with Exceptions.

* **Reentrance Vulnerabilities**

And finally we are here to talk about Reentrance Vulnerabilities. As long as you haven't been living under a rock for the past couple of years, you would have heard of the [DAO exploit](https://blog.ethereum.org/2016/06/17/critical-update-re-dao-vulnerability/) which happenned on 17th July, 2016 and robbed the DAO platform, hosted on top of Ethereum, of 60 million dollars! The DAO attack is a classic example of exploiting the reentrance vulnerabilities prevalent in the semantics of smart contracts. Let us take some time to study it: 

```javascript
1 contract SendBalance {
2   mapping (address => uint) userBalances;
3   bool withdrawn = false;
4   function getBalance(address u) constant returns(uint){ 
5       return userBalances[u];
6   }
7   function addToBalance() {
8       userBalances[msg.sender] += msg.value;
9   }
￼￼￼￼￼￼￼￼￼10  function withdrawBalance (){ 
11  if (!(msg.sender.call.value(
12      userBalances[msg.sender])())) { throw; } 
13  userBalances[msg.sender] = 0;
14}}
```
In Ethereum, when a contract calls another the caller contract is stuck in an intermediate state. So if you observe Line 11, this contract actually calls another contract and only then changes the value to 0. Now imagine a case where this contract is in the middle of the call made at line 11 and in the meantime another contracts(assume another for now) calls the 'stuck state' contract. Well that reaches line 11 and again gets stuck. In a similar way imagine this cycle of 'reenter and call again' goes on till you exhaust all the gas. Well at this point finally line 13 gets executed and this entire chain of reentrance unravels zeroing out the total balance held by the contract. And that ladies and gentlemen is the DAO attack. Reentrance has been a well studied topic in Concurrency. But this is inherently sequential code? You ask what has this got to do with concurrency. As an answer I would like to quote a line from the paper by Sergey, et al
```
Previous analyses of this bug have indicated that the problem is 
due to recursion or unintended reentrancy. In a narrow sense this 
is true, but in a wider sense what is going on is that sequential 
code is running in what is in many senses a concurrent environment.
```

While this paper draws an analogy between smart contracts and problems studies in traditional lock based concurrency, it also notes that the locking contract pattern has a significant non-modular design. Also lock based concurrency mechanisms are traditionally not composable. As a result of which we will look at an alternative to lock based concurrency which is known as Transactional Memory.

**TRANSACTIONAL MEMORY**

Transactional Memory as a concept is fairly simple. Imagine you have the following block of code:
```javascript
atomic {
    <Your code goes here>
}
```
And every code that you write inside this block is executed *atomically*. It is *isolated* from any other form of interaction that might be going around in other threads. And at the end of this all threads are guranteed to see a *consistent* view of the main memory. Sounds similar to ACID gurantees in a database? You are correct. Transactional Memory APIs are extremely simple to understand, however the machinery operating underneath it might be a little complex. It varies from implementation to implementation. The most popular is the Transactional Locking II algorithm as stated in [this paper](http://www.disco.ethz.ch/lectures/fs11/seminar/paper/johannes-2-1.pdf) by Dice, et al. This is a must read paper. However as we are interested in the *lazy* version of STM we will be working with the implementation provided by the paper on [Composable Memory Transactions](http://simonmar.github.io/bib/papers/stm.pdf) by Harris, et al.

Haskell provides something called a `TVar` wherein you encapsulate you state. The APIs provided for operating on this `TVar` are fairly simple:
```haskell
data TVar a
newTVar   :: a -> STM(TVar a)
readTVar  :: TVar a -> STM a
writeTVar :: TVar a -> a -> STM ()
``` 

The `STM` you see here in the type signature is a Monad instance. Its APIs are also fairly intuitive
```haskell
data STM a
instance Monad STM

atomically :: STM a -> IO a
retry      :: STM a
orElse     :: STM a -> STM a -> STM a

throw :: Exception -> STM a
catch :: STM a -> (Exception -> STM a) -> STM a
``` 

We are primarily interested in the `atomic`

[1]A transaction is something which results in the execution of a smart contract.