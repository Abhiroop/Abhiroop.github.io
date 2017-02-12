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
You can see most of the update, add or cons operations have logarithmic time complexities. And that runs very well for most of our day to day applications. For instance persistent hash maps in Clojure have `O(log n)` lookup cost. In fact it is base 32 logarithm so that nearly translates to `O(1)`.

So now we are going to try and implement this amazing data structure in Haskell. Well finger trees are not exclusive to lazy languages. However, it is heavily based on one of the most fundamental typeclasses of Haskell: Monoids and also it makes essential use of laziness but it is suitable for strict languages that provide a lazy evaluation primitive. So without any further delay let us dive into Finger Trees.

The definition is very similar to a normal recursive tree definition in Haskell
```haskell
data Tree v a = Leaf   v a | 
                Branch v (Tree v a) (Tree v a)
```
This is a binary tree. However we store the values **only at the leaves** unlike a normal BST. We will majorly optimize the structure so that the access of the leaves happens in logarithmic time. 

Now the point of interest for us here is the type parameter `v`. The metadata in `v` gives us information on how to traverse the tree. However we want this to be as generic as possible, only then can we use the same underlying structure to answer multiple types of queries. We are going to take 2 examples to compare and contrast how to unify the 2 metadata: 
1. A list (linked list to be precise) with logarithmic random access.
2. A priority queue with logarithmic peek.

We know a normal `xs!!n` operation on a list in any programming language runs at `O(n)`. However we will use a simple trick of annotating each subtree with its **size**. The size of a tree is the number of nodes in the tree.Hence the metadata of a tree will look like this:
```
         4
       /   \
      2     2
     / \   / \
    1   1 1   1
```
And of course the leaves will contain the actual value. So whenever we are asked to access the nth element we do this: 
```
if n < size(left child node) 
    go left and look for n
else
    go right and look for n - size(left child node)
Break recursion at the leaf node. 
```  
Take some time and evaluate this approach.

Before starting with the actual code lets think about the priority queue to. We will model the priority queue like a [tournament tree](https://en.wikipedia.org/wiki/Selection_algorithm#Tournament_Algorithm). Here lets represent the priority using integral numbers. The lower the number, the higher the priority. So if we model the same using our Finger Tree the type parameter `v` will now represent the minimum of its 2 subtrees. Hence:
```
        3
      /   \
     5     3
    / \   / \
   8   5 7   3
```
Thats a simple tournament tree with winning strategy being the function `min`, which leads to the super simple recirsion:
```
if priority(node) = priority (left child)
    recurse down left child
else
    recurse down right child
Break recursion at the leaf node of course!!
```
Thats laughably simple. Right? The trick is unifying the above 2 metadata functions for priority queue and random access list. And the grand unifier of it all is the elegant Monoid typeclass.

A dull definition of a monoid is a `a binary associative operation with an identity.` However if we look at the typeclass definition of Monoid:
```haskell
class Monoid a where
  mempty :: a
  mappend :: a -> a -> a
```
While the `mempty` operation is simply the identity of the monoidal structure, `mappend` is an incredibly useful abstraction. Although the first thing that comes to our mind when thinking about "appending" is `++` because we are too used to operating with lists. But in a general sense a `mappend` operation is a way to summarize about the data structure. If you come to think of it **most search problems deal with searching for a specific element within a complex structure. If that structure happens to be an instance of Monoid you can use the same `mappend` operation to _summarize_ and answer queries on that very same data structure**. The `mappend` operation is represented as `<>`. So, the above trees should be able to answer:
```
     (v1<>v2) <> (v3<>v4)         
            /    \                  
           /      \               
          /        \              
      v1 <> v2  v3 <> v4              
        /  \      /  \                 
       v1  v2    v3  v4                    
       a1  a2    a3  a4     
```
And owing to the associativity property of monoids the above works for any amount of skewedness of the tree!

Now to code. Let us first capture the different metadata types for the 2 separate structure:
```haskell
newtype Size = Size {getSize :: Int} -- for the random access list

newtype Priority = Priority {getPriority :: Int} -- for the priority queue
```
Assuming we have figured out a generic function `tag` which given a `Branch` returns the metadata for that particular branch. In short:
```haskell
  tag :: Tree t a -> t
```
We can capture this essence very simply, introducing a typeclass called `Tag` which dispatches the `tag` function depending on the type of metadata i.e `Size` or `Priority`. Hence:
```haskell
class Tag t where
  tag :: Tree t a -> t

instance Tag (Tree Size a) where
  tag (Leaf v _)     = v
  tag (Branch v _ _) = v

instance Tag (Tree Priority a) where
  tag (Leaf v _)     = v
  tag (Branch v _ _) = v 
```

Now that we have the above at our disposition let us quickly make `Size` and `Priority` instances of Monoid. So,
```haskell
instance Monoid Size  where
  mempty  = Size 0
  Size x `mappend` Size y = Size (x + y)

instance Monoid Priority where
  mempty  =  Priority (maxBound :: Int)
  Priority x `mappend` Priority y = Priority (min x y)

```
Now this gives me the amazing power to define by branch function as a smart constructor like this:
```haksell
branch :: (Tag v, Monoid v) => Tree v a -> Tree v a -> Tree v a
branch x y = Branch (tag x <> tag y) x y
```

And for the leaf it is dead simple to get the tag from the element inserted. We will capture this behavior in a multi parameter typeclass(using a GHC extension):
```haskell
class Monoid v => Measured v a where
    measure :: a -> v

instance Measured Size Int where
    measure _ = Size 1           -- the leaf elemnt of Size 1

instance Measured Priority Int where
    measure a = Priority a       -- priority of the element as entered

instance (Tag v, Measured v a) => Measured v (Tree v a) where
    measure = tag

leaf :: Measured v a => a -> a -> Tree v a
leaf a priority = Leaf (measure priority) a
```
And thats it! You have your finger tree defined. As you had more operations and extend it to answer more type of queries instantiate the `Monoid` typeclass and figure out ways of combining the elements into action. Have a look at the random access function and the priority queue lookup function together. They are just super simple Haskell implementations of the pseudocode deifned above:
```haskell
(!!!) :: Tag Size => Tree Size a-> Int -> a
(Leaf _ a)     !!! 0 = a
(Branch _ x y) !!! n
  | n < getSize (tag x) = x !!! n
  | otherwise           = y !!! (n - getSize (tag x)) 

winner :: Tag Priority => Tree Priority a -> a
winner t = go t
   where
   go (Leaf _ a) = a
   go (Branch _ x y)
     | getPriority (tag x) == getPriority (tag t) = go x
     | getPriority (tag y) == getPriority (tag t) = go y
```

I have also implemented the search functionality in my [github repo](https://github.com/Abhiroop/HaskAl) which actually unifies the above 2 functions into an even higher abstraction, but the above examples should drive the thought to your head.

The above implementations from Heinrich Apfelmus' amazing blog on Finger Trees. However the code in his blog does not type check. I believe he wanted to expose the ideas majorly. I made sure the code mentioned here does type check with GHC 7.10 and it actually required some effort to fix the types. And adding a new typeclass actually did modify the type signature quite a bit and those signatures will tell you a lot about the relation between these structures. You can go through the code [here](https://github.com/Abhiroop/HaskAl/blob/master/FingerTree.hs). I urge interested readers to actually study the [original paper](http://www.staff.city.ac.uk/~ross/papers/FingerTree.pdf) by Ralf Hinze and Ross Patterson and it actually happens to be one of the more accessible papers on functional programming and Haskell.