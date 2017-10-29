---
layout: post
title: Persistent Red Black Trees in Haskell
---

While Haskell is steadily gaining mainstream adoption in the industry, it still remains one of the most viable languages used as a teaching medium. Even if we strip the fancy type level features in Haskell, algebraic data types and pattern matching are quite expressive enough to represent a lot of ideas.

In this post I will be looking at the construction and operations of Red Black trees. Of special interest here would be the deletion operation, as the operation of `delete` is inherently opposed to Haskell's fundamentals of immutability. We will look at the various operations as creating a new version of the tree rather than mutating an existing version, which gives the persistent quality to our data structure.

This post doesn't use any fancy type level features of Haskell(except at the very end) and assumes only basic familiarity with the language. However the deletion operation is quite involved and requires the reader to be fairly meticulous so as to not miss any of the possible cases.

A red black tree is a type of a binary search tree with the ability to self balance itself. This property of self balancing is highly desirable as a plain binary search tree reduces to `O(n)` worst case time complexity for `search`, `insert` and `delete` operations. The balancing nature of red black tree gives it a worst case time complexity of `O(log n)`. It is not truly balanced in the sense that the heights of the various subtrees can differ, but the height of the longest subtree would be a maximum of `2log(n+1)`, which effectively gives it a balanced nature.

From a practical perspective, it is not expected of you to code out an implementation of red black tree every other day but it is helpful to know the implementations and understand the trade-offs made so that performance analysis on some critical code can be made. Notably [TreeMap](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/8u40-b25/java/util/TreeMap.java) from Java collections is implemented using a Red Black Tree(however it is not a persistent implementation). In practise most implementations of `maps`, `sets` and other useful structures are implemented using balanced binary search trees. Hence without any further ado lets jump into the datatypes.

------------------------------------------------------

Defining the tree
-----------------

We will start by defining 2 colors red and black and use that as a metadata in our actual tree type.

```haskell
data Color = R | B deriving Show

data Tree a = E | T Color (Tree a) a (Tree a) deriving Show
```

Now for a red black tree to be balanced it needs to follow a set of invariants. These invariants technically can be encoded into the Haskell type system but to keep the implementation simpler we define it using functions and verify them at runtime. The invariants are:

```
1. No red node has a red parent or a red node has only black children.
2. Every path from the root node to an empty node contains the same number of black nodes.
3. The root and leaves of the tree are black.
```

Take some time to internalize these invariants, as they should not be broken under any circumstances. We can relax some of them locally in certain cases but we make sure some other functions at the global level handles the violation and rectifies it. We are going to start by implementing the `member` function which basically tells us if the element belongs to the tree or not. This is not very different from a regular `BST`.

```haskell
member :: (Ord a) => a -> Tree a -> Bool
member x E    = False
member x (T _ a y b)
  | x < y     = member x a
  | x == y    = True
  | otherwise = member x b
```

This should be self explanatory. If we find the element we report `True` otherwise we recurse down the left or right subtree.

------------------------------------------------

Insertion
---------

Now lets look at one of the more involved operations: `insert`.

The insertion operation was described by Chris Okasaki in his classic book "Purely Functional Data Structures". The insertion of a node is colored red in the beginning so as not to affect the height. However this might end up violating the first invariant. To restore the first invariant we have to balance the tree recursively. The function looks something like this:

```haskell
insert :: (Ord a) => a -> Tree a -> Tree a
insert x s = makeBlack $ ins s
  where ins E  = T R E x E
        ins (T color a y b)
          | x < y  = balance color (ins a) y b
          | x == y = T color a y b
          | x > y  = balance color a y (ins b)
        makeBlack (T _ a y b) = T B a y b
```

The points to notice here in this definition are the functions `makeBlack` and `balance.` Minus those functions the insertion is exactly similar to an insert in a `BST`. The `makeBlack` function is also a fairly simple function, given a Node it changes the color of the node to black irrespective of the node's color. The purpose of this function is that after applying `balance` recursively the final tree might have 2 consecutive red nodes at the top of the tree. This would be a violation of invariant 1. By blackening the root we restore invariant 1 as well as we invariant 2 stays intact, as only the root of the tree gets colored. A more opt name for the function should have been `blackenRoot` perhaps. However this is quite simple.

The trick is now writing and understanding the `balance` function. Now we know  that the insertion might have violated invariant 1 and as a result of which it might have created a tree with 2 consecutive red nodes. So all we have to think is given the original balanced tree with no violations what are the possible ways in which a red node might have been inserted which breaks the invariant 1. Let us see:
Due to lack of a red pen I am using a blue pen to represent red nodes. So this is technically a blue black tree. But you get the point:

Figure

Now algebraic data types and pattern matching makes it very easy to express each case. So if we write out the tree structure as demonstrated in the figure the 4 cases would look like this:

```haskell
T B (T R (T R a x b) y c) z d  
T B (T R a x (T R b y c)) z d  
T B a x (T R (T R b y c) z d)  
T B a x (T R b y (T R c z d)) 
```

So all we have to do is find a way to balance each of the cases. As it is given in the figure, the solution to balancing each of the 4 cases is the exact same. By using this transformation the tree restores the invariant 1 and invariant 2 stays intact too(Invariant 3 is almost always intact and easiest to enforce). So what happens for the other configurations of the tree? We return the nodes intact. So writing the same in Haskell:

```haskell
balance :: Color -> Tree a -> a -> Tree a -> Tree a
balance B (T R (T R a x b) y c) z d = T R (T B a x b) y (T B c z d)
balance B (T R a x (T R b y c)) z d = T R (T B a x b) y (T B c z d)
balance B a x (T R (T R b y c) z d) = T R (T B a x b) y (T B c z d)
balance B a x (T R b y (T R c z d)) = T R (T B a x b) y (T B c z d)
balance color a x b = T color a x b
```

Languages like OCaml and SML support something called `OR patterns` which make it even simpler to write the function definition where multiple patterns have the same answer. And in fact there is a [GHC proposal](https://github.com/ghc-proposals/ghc-proposals/pull/43) in progress about adding OR patterns to Haskell. This implementation is much more readable, expressive and most importantly intuitive compared to any other imperative language implementations of the same idea. Writing a red black tree balancing algorithm is a big deal in such languages but with ADTs and pattern matching it literally begs to follow the diagram given above.

-------------------------------------------------------------

Deletion
--------

Now moving on to the `delete` operation. This one is lot more involved and we will do it step by step.

First a deletion followed by any kind of balancing has a possibility of bubbling a red node to the top, so we need the `makeBlack` function that we used in the `insert` function, followed by that we can call an auxiliary `del` function which will effectively delete the node and balance the tree. In Haskell:

```haskell
delete :: (Ord a) => a -> Tree a -> Tree a
delete x t = makeBlack $ del x t
  where makeBlack (T _ a y b) = T B a y b
```

This is fairly simple.