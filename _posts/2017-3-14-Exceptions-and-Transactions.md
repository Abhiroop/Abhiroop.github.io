---
layout: post
title: Understanding the Transactional Nature of Smart Contracts
---
In this post I attempt to summarize my understandings about the transactional nature of Smart contract execution. I conducted this study, while trying to understand the DAO exploit. The basis of the DAO exploit is a recursive(or rather reentrant) message call. One important point to note that an exception(during a message call or otherwise) in the Ethereum Virtual Machine , would imply reverting all changes to **state and balance**. Solidity has a documentation on [cases](http://solidity.readthedocs.io/en/latest/control-structures.html#exceptions) where exceptions are thrown automatically. However in certain cases like `send` we need to manually raise an exception using `throw`. So, the attacker has to be careful not to hit any exceptions while attempting the reentrant call to avoid reverting. I also rectify certain mistaken assumptions, I made in my previous post.

So let us start by looking at the simplified version of a reentrant bug, as mentioned in multiple blogs and the paper Making Smart Contracts Smarter.

```javascript
1 contract SendBalance {
2 mapping (address => uint) userBalances;
3 bool withdrawn = false;
4 function getBalance(address u) constant returns(uint){ 
5   return userBalances[u];
6 }
7 function addToBalance() {
8   userBalances[msg.sender] += msg.value;
9 }
￼￼￼￼￼￼￼￼￼10 function withdrawBalance (){ 
11    if (!(msg.sender.call.value(
12          userBalances[msg.sender])())) { throw; } 
13    userBalances[msg.sender] = 0;
14 }}
```
The code above clearly demonstrates a coding anti pattern in line 11 and 13 where the receiver is sent the amount in line 11 and his balance is zeroed out later, whereas clearly we should have coded it the other way around. However, please do note this is not the DAO contract. This is a much simpler contract and had the DAO contract been written this way, *the sent amount would have been reverted, if the `call.value` ran into an exception*. I will expand upon this statement in a bit.

One of my principal sources of confusion was the rollback mechanism of the EVM and whether rolling back would result in reverting the balance to the sender. And is sending money equivalent to a *side effect*? 

Well, an expression or a function is a side effect if it modifies some state outside its *scope* or has an observable interaction with the outside world. In this case if we take a look at the operational semantics of the EVM in the [Ethereum yellowpaper](http://gavwood.com/paper.pdf):

![an image alt text]({{ site.baseurl }}/images/semantic.png "EVM SEMANTICS")

As we can see above, whenever an exception occurs the *world state* σ should be reverted to the point prior to intermediate state σ'. Now the question is what is σ? And does it capture the account balance within its scope? Well of course. According to the yellowpaper, σ is used to denote the *World State*, which is a mapping between addresses and account states. Implementations of the paper, models this mapping as a `Merkle Patricia Trie`. And being an immutable structure it allows any previous state (whose root hash is known) to be recalled by simply altering the root hash accordingly. 

So rolling back would simply involve altering the root hash of the Merkle Patricia trie. Hence if an Account A owns 50$ and Account B owns 100$. And while attempting to send 5$ from Account A to Account B an exception is thrown(manually using `throw`), the root hash of the trie would be reverted to the one where A and B owns 50$ and 100$ respectively. Interested readers can look at the model of the State, in the C++ implementation of Ethereum [here](https://github.com/ethereum/cpp-ethereum/blob/6f0c62e759fe9c950dbd481c1514f869bdd70a93/libethereum/State.h).

Now, let us confirm the same fact from the [documentation of Solidity](http://solidity.readthedocs.io/en/latest/control-structures.html#exceptions), which states:
```
The effect of an exception is that the currently executing call 
is stopped and reverted (i.e. all changes to the state and balances 
are undone) and the exception is also “bubbled up” through Solidity 
function calls (exceptions are send and the low-level functions call, 
delegatecall and callcode, those return false in case of an exception).
```
 The above is pretty self explanatory. Let us verify the above with a Contract of our own. There is a lot of documentation out there about setting up an ethereum client and running a full node or a test node. I preferred to use `geth` for my experiments. Also for simplification of the process of deploying your contracts and playing around with them, you can use [testrpc](https://github.com/ethereumjs/testrpc). Testrpc comes with 10 test accounts. For deployment and contract development in general I would suggest people to use the [Truffle framework](http://truffleframework.com/) which does the compilation, linking, deployment and binary management. Also for contract programmers from India, Kraken and Coinbase does not trade ethers in India currently. You can purchase ethers from [Ethex India](https://ethexindia.com/). I will be using a very simple token contract as mentioned in the examples of `geth` with slight modifications:

 ```javascript
pragma solidity ^0.4.9;
contract token { 
    mapping (address => uint) public coinBalanceOf;
    

  /* Initializes contract with initial supply tokens to the creator of the contract */
  function token(uint supply) {
        coinBalanceOf[msg.sender] = supply;
    }

  /* Very simple trade function */
    function sendCoin(address receiver, uint amount) returns(bool sufficient) {
        if (coinBalanceOf[msg.sender] < amount) return false;
        coinBalanceOf[msg.sender] -= amount;
        coinBalanceOf[receiver] += amount;
        if (!receiver.send(amount))
            throw;
        return true;
    }
}
 ```

 The `geth` tutorial covers everything about setting up your account. The modification which I have made is removing the event `CoinTransfer` and instead sending the coin using `send` and actually checking if an exception occurs during the `send`, in that case I explicitly `throw`. This is an additional statement to be executed before deploying the contract:
 ```
 personal.unlockAccount(web3.eth.accounts[0], "<Your password here>")
 ```
Upon compiling the contract, the solc online compiler will give you the gas estimates. In this case:
```
Creation: 20341 + 122600
External:
  coinBalanceOf(address): 353
  sendCoin(address,uint256): unknown
```

So let us load our account with less gas so that the `sendCoin` can fail and an exception be thrown. With the basic initialization in place and following the documentation, when we call the `sendCoin` function, we encounter an exception. If we were to inspect the state of `eth.accounts[1]`(the receiver) it still wouldn't have any token and the entire supply would reside with `eth.accounts[0]`(the sender), which implies a proper rollback takes place. Please do note that if you go ahead with this, the gas will end up being consumed.

So, having done this experiment, if we return to look at the code, I mentioned at the beginning of my post, the vulnerable function looks like this:
```javascript
1 function withdrawBalance (){ 
2    if (!(msg.sender.call.value(
3          userBalances[msg.sender])())) { throw; } 
4    userBalances[msg.sender] = 0;
5 }}
```

If this was indeed the DAO contract, the moment `call.value` would return `false`, `throw` would be called which raises an exception and the entire transaction would be reverted to its old state. Also it is a bad idea to use `call.value` in place of `send`. Because `send` by default doesn't forward any gas to the receiver. However `call.value` does that and would allow the receiver to make reentrant calls without burning any gas for the message call in between. Please note, if I had not manually checked for the `send` to return a `false` no exception would occur and the balance would end up getting transferred despite of running out of gas. We can conduct an experiment on the same by deploying the Token contract using the `CoinTransfer` event as mentioned in the `geth` docs. The key takeaway for a contract developer is this:

1. **The concept of rollback is tied to an exception. Exceptions results in rollback.**
2. **The `send` function of Solidity returns `false` on facing an exception. You have to manually check and use a `throw` statement to trigger the rollback.**

For those interested to study the DAO contract in detail it can be found over here: [The original DAO contract](https://github.com/slockit/DAO/blob/DAO11/DAO.sol#L738). Phil Daian and Peter Vessenes have done a very good job of explaining the DAO attack line by line in their respective blogs. I would urge the interested reader to head there to understand the DAO exploit in depth, which deserves a separate post on its own. The only thing I would like to point out is the most vulnerable part in that contract:
```javascript
1 // Burn DAO Tokens
2 Transfer(msg.sender, 0, balances[msg.sender]);
3 withdrawRewardFor(msg.sender); // be nice, and get his rewards
4 totalSupply -= balances[msg.sender];
5 balances[msg.sender] = 0;
6 paidOut[msg.sender] = 0;
7 return true;
```
Of course the anti-pattern of 'sending the amount first and zeroing out later' is visible in line 3 onwards. Another critical bug as pointed by Peter Vessenes is in line 2, where the `Transfer` function is called instead of `transfer`(Note the capitalization). The `transfer` function would reduce the user balance before the vulnerable withdraw, hence the call should have looked more like 
```javascript
if (!transfer(0 , balances[msg.sender])) { throw; } 

instead of

Transfer(msg.sender, 0, balances[msg.sender]);
```

---------
---------

**PART 2 : INTERNALS OF THE ROLLBACK MECHANISM**

[Note: The following sections might not be of much utility if you are looking for advice on writing contracts. They principally document my excursion into the EVM codebase and might contain some incorrect or incomplete information, as I have just started researching and understanding the Ethereum codebase.]