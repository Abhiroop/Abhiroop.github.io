---
layout: post
title: Understanding the Transactional Nature of Smart Contracts during message calls
---
In this post I attempt to summarize my understandings about the transactional nature of Smart contract execution while attempting message calls. I conducted this study, while trying to understand the DAO exploit. The basis of the DAO exploit is a recursive(or rather reentrant) message call. One important point to note that an exception(during a message call or otherwise) in the Ethereum Virtual Machine , would imply reverting all changes to **state and balance**. So, the attacker has to be careful not to hit any exception while attempting the reentrant call. I also rectify certain mistaken assumptions, I made in my previous post.

So let us start by looking at the simplified version of of a reentrancy bug, as mentioned in multiple blogs and the paper Making Smart Contracts Smarter.

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
The code above clearly demonstrates a coding anti pattern in line 11 and 13 where the sender is sent the amount in line 11 and his balance is zeroed out later,whereas clearly we should have coded it the other way around. However, please do note this is not the DAO contract. This is a much simpler contract and had the DAO contract been written this way, *the sent amount would have been reverted, if the `call.value` ran into an exception*. I will expand upon my previous statemnet now.