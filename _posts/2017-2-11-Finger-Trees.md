---
layout: post
title: Finger Tree - The ultimate data structure?
---

In this post I will be talking about one of the most useful data structures I have come across in quite some time - Finger Trees.

Why do I ask if its the ultimate data structure? Well read this excerpt from the the [original paper](http://www.staff.city.ac.uk/~ross/papers/FingerTree.pdf) on Finger Trees:

```
a functional representation of persistent sequences supporting access to the ends in amortized constant time, 
and concatenation and splitting in time logarithmic in the size of the smaller piece. Further, by defining 
the split operation in a general form, we obtain a general purpose data structure that can serve as a sequence, 
priority queue, search tree, priority search queue and more.
```
Need I say more? A data structure so multi faceted that it can, at the same time, function as multiple other structures answering various types of search queries in logarithmic time.
To be honest, I see finger trees more as a substrate to build other data structures on top of it. They can be used to implement nearly all the abstract data structures mentioned in [Okasaki's classic book](http://www.cs.cmu.edu/~rwh/theses/okasaki.pdf). Take a look:
![an image alt text]({{ site.baseurl }}/images/datastructure.png "Functional data structures")
You can see most of the update, add or cons operations have logarithmic time complexities. And that runs very well for most of our day to day applications. For instance persistent hash maps in Clojure have O(log n) lookup cost. In fact it is base 32 logarithm so that nearly translates to O(1).

So now we are going to try and implement this amazing data structure in Haskell. Well finger trees are not exclusive to lazy languages. However, it is heavily based on one of the most fundamental typeclasses of Haskell: Monoids and also it makes essential use of laziness but it is suitable for strict languages that provide a lazy evaluation primitive. So without any further delay let us dive into Finger Trees.

The definition is very similar to a normal recursive tree definition in Haskell
```haskell
data Tree v a = Leaf   v a | 
                Branch v (Tree v a) (Tree v a)
```
This is a binary tree. However we store the values **only at the leaves** unlike a normal BST. We will majorly optimize the structure so that the access of the leaves happens in logarithmic time. 

Now the point of interest for us here is the type parameter `v`. The metadata `v` gives us information on how to traverse the tree. However we want this to be as generic as possible, only then can we use the same underlying structure to answer multiple types of queries. We are going to take 2 examples to compare and contrast how to unify the 2 metadata: 
1. A list (linked list to be precise) with logarithmic random access.
2. A priority queue with logarithmic peek.

The above implementation is based on Heinrich Apfelmus' blog on Finger Trees. However the code in his blog does not type check. I believe he wanted to expose the ideas majorly. I made sure the code mentioned here does type check with GHC 7.10 and it actually required some(not a lot ;)) effort to fix the types. And adding a new typeclass actually did modify the type signature quite a bit and those signatures will tell you a lot about the relation between these structures. You can go through the code [here](https://github.com/Abhiroop/HaskAl/blob/master/FingerTree.hs). I urge interested readers to actually study the [original paper](http://www.staff.city.ac.uk/~ross/papers/FingerTree.pdf) by Ralf Hinze and Ross Patterson and it actually happens to be one of the more accessible papers on functional programming and Haskell.