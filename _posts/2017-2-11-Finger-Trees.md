---
layout: post
title: Finger Tree - The ultimate data structure?
---

In this post I will be talking about one of the most useful data structures I have come across in quite some time - Finger Trees.

Why do I ask if its the ultimate data structure? Well read this excerpt from the the original paper[1] on Finger Trees:

```
a functional representation of persistent sequences supporting access to the ends in amortized constant time, 
and concatenation and splitting in time logarithmic in the size of the smaller piece. Further, by defining 
the split operation in a general form, we obtain a general purpose data structure that can serve as a sequence, 
priority queue, search tree, priority search queue and more.
```
Need I say more? A data structure so multi faceted that it can, at the same time, function as multiple other structures answering search queries in logarithmic time.
To be honest, I see finger trees more as a substrate to build other data structures on top of it.
![an image alt text]({{ site.baseurl }}/images/datastructure.png "an image title")

[1] http://www.staff.city.ac.uk/~ross/papers/FingerTree.pdf