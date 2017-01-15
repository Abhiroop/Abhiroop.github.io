---
layout: post
title: Powerset in Haskell
---

A power set is simply defined as *all the subsets of a set*.

So for a given set the number of possible subsets are `2^n`.

Example:

Given a set `S = {a,b,c}`.

The powerset is defined as:

`P(S) = { {}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c} }`.

So now we are tasked with finding out the powerset for a given set of elements.

In a general imperative approach we would define the powerset in any imperative language (like Java) like this:

```java
public static <T> Set<Set<T>> powerSet(Set<T> originalSet) {
    Set<Set<T>> sets = new HashSet<Set<T>>();
    if (originalSet.isEmpty()) {
        sets.add(new HashSet<T>());
        return sets;
    }
    List<T> list = new ArrayList<T>(originalSet);
    T head = list.get(0);
    Set<T> rest = new HashSet<T>(list.subList(1, list.size())); 
    for (Set<T> set : powerSet(rest)) {
        Set<T> newSet = new HashSet<T>();
        newSet.add(head);
        newSet.addAll(set);
        sets.add(newSet);
        sets.add(set);
    }       
    return sets;
}  
```

Before you start reading and understanding the bits and nuances of the above snippet, I would like 
to stress on the point that while this code is algorithmically correct, looking at it doesn't give you any information about the 
definition of a powerset. It represents the thought process that would go through your mind while
building the powerset one by one at a time.

While this approach works well for most people, declarative languages use a separate way of expressing this problem. It is more of a top
down approach of expressing your problem statement and defining it in terms of functions which recursively generates the answers for you.

In Haskell the solution to the same question looks like this:

```haskell
powerset = filterM (const [True,False])
```

And thats it!

The main motivation of this article is to explain what is going on above, as it might not be immediately apparent that how is the recursive machinery operating underneath. But what a beautiful and elegant looking function!!

First of all this is written in point free style Haskell.

The way to understand this definition is to study the source code of `filterM`.

But before that lets attempt to define `filterM` ourselves:
```haskell
import Control.Monad

filterM' :: (Monad m) => (a -> m Bool) -> [a] -> m [a]
filterM' p [] = return []
filterM' p (x:xs) =
    let rest = filterM' p xs in
      do b  <- p x
         if b then liftM (x:) rest
              else            rest
```

This definition is different from the real implementation of `filterM`.

But works absolutely fine. Lets test it out:

```haskell
λ> filterM' (const [True,False]) [1,2,3]
[[1,2,3],[1,2],[1,3],[1],[2,3],[2],[3],[]]
```

Before going through it step by step lets us understand the 2 most important catches in the definition of `filterM`.

First for beginners who haven't used the `do` notation in monads.

This step:
```haskell
b <- p x
```
when applied, will result in `[True,False]`. In the first iteration `b` assumes the value of `True` and then `False` and followed by concatting the results of the two. The definition of `bind` for the List monad will help you understand more:

```haskell
instance Monad [] where  
    return x = [x]  
    xs >>= f = concat (map f xs)  
    fail _ = []  
```

The second catch is `liftM`

`liftM` basically promotes a function to a Monad. Super simple definition of `liftM` below:
```Haskell
liftM   :: (Monad m) => (a1 -> r) -> m a1 -> m r
liftM f m1              = do { x1 <- m1; return (f x1) }
```

Taking this out for a small spin:
```haskell
λ> liftM ('h':) ["ello","ey","i","otel"]
["hello","hey","hi","hotel"]
```

And now if you take a look at the definition of `filterM`, the entire definition will not look very differnt from the definition of plain old `filter` except for the lifting of Monads bit.

Still to simplify your understanding I am going to explain the `filterM` monad step by step using a small list `[1,2,3]`

So,
```
first x=1, xs=[2,3]
       recursion x=2, xs=[3]
              recursion x=3, xs=[]
                   final recursion x=[]
                   function returns [[]]
              now x=3 and rest=[[]]
              liftM (3:) [[]]  ++  [[]]
              returns[[3],[]]
        now x=2 and rest = [[3],[]]
        liftM (2:) [[3],[]]  ++   [[3],[]]
        returns [[2,3],[2],[3],[]]
now x=1 and rest = [[2,3],[2],[3],[]]
liftM (1:) [[2,3],[2],[3],[]] ++ [[2,3],[2],[3],[]]
returns [[1,2,3],[1,2],[1,3],[1],[2,3],[2],[3],[]]
```

This is how the call stack would proceed and end up constructing the powerset for you.

Keep in mind this is not just another functional construct but a testament to the beauty and elegance you can attain while writing in Haskell. In most other functional languages including Clojure it would not be possible
to express this operation so succintly. Even in Scala you would have to resort to Scalaz to express this operation. This is owing to the flexibility of the do notation in Haskell, and while other languages might support monads but Haskell syntax makes it all the more fun and idiomatic to use!
