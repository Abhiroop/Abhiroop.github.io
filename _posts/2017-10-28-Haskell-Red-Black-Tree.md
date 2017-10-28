---
layout: post
title: Persistent Red Black Trees in Haskell
---

While Haskell is steadily gaining mainstream adoption in the industry, it still remains one of the most viable languages used as a teaching medium. Even if we strip the fancy type level features in Haskell, algebraic data types and pattern matching are quite expressive enough to represent a lot of ideas.

In this post I will be looking at the construction and operations of Red Black trees(hereby referred to as RB Tree). Of special interest here would be the deletion operation, as the operation of `delete` is inherently opposed to Haskell's fundamentals of immutability. We will look at the various operations as creating a new version of the tree rather than mutating an existing version, which gives the persistent quality to our data structure.

This post doesn't use any fancy type level features of Haskell(except at the very end) and assumes only basic familiarity with the language. However the deletion operation is quite involved and requires the reader to be fairly meticulous so as to not miss any of the possible cases.


blah blah about RB TREE
an while it is not expected of you to code out an implementatiions of red black tree after a friday scrum. it is helpful to know the implementations and understand the tradeoffs made so performance analysis when eroting crticcal code is eadu