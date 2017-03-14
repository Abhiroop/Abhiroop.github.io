---
layout: post
title: Understanding the Transactional Nature of Smart Contracts during message calls
---
In the post I attempt to summarize my understandings about the transactional nature of Smart contract execution while attempting message calls. I conducted this study, while trying to understand the DAO exploit. The basis of the DAO exploit is a recursive(or rather reentrant) message call. One important point to note that an exception(during a message call or otherwise) in the Ethereum Virtual Machine , would imply reverting all changes to **state and balance**. So, the attacker has to be careful not to hit any exception while attempting reentrant call. I also rectify certain mistaken assumptions, I made in my previous post.