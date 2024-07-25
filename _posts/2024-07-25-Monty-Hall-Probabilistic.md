---
layout: post
title: Solving Monty Hall with the List monad (or Probabilistic Programming)
---

Continuing from my [Bayes Theorem-based blog post](https://abhiroop.github.io/Monty-Hall/) on solving the Monty-Hall problem, I will now show a rather elegant way to model the solution that exploits
the humble List monad. The goal is to help the contestant answer the question:*should I switch my choice?*, when the contestant has already chosen a door, followed by Monty Halls opening a different door and 
asking the question. This post was inspired by a [less-than 100 lines implementation of a probabilistic programming monad](https://dennybritz.com/posts/probability-monads-from-scratch/) in Haskell 

We will begin by representing discrete distributions as a Haskell datatype:

```Haskell
type Prob = Double -- probability of an outcome

newtype Dist a = Dist { unpackDist :: [(a, Prob)] }
```

This is a nicer reformulation of distributions that could be simply modelled by the List monad. For example, in an experiment with 5 tosses (modelled as `data Toss = Head | Tail`), that leads to 
4 Heads and 1 Tail, our `Dist` type will model this as `Dist [(Head, 0.8), (Tail, 0.2)]`. However, we could have simply modelled this as a plain list as `[Head, Head, Head, Head, Tail]`. It is simply the
`Dist` type is nicer to work with (but it is simply a wrapper on the plain `List`).

Next, to compute probabilities accurately, its easier for our representation if we normalise the distribution, we can simply do that using:

```Haskell
normP :: [(a, Prob)] -> [(a, Prob)]
normP xs = map (\(x, y) -> (x, y / (sumP xs))) xs
  where
    sumP :: [(a, Prob)] -> Prob
    sumP = sum . map snd
```

Given the above, we can now define the uniform distribution:

```Haskell
uniform :: [a] -> Dist a
uniform = Dist . normP . map (, 1.0)
```

```Haskell
> uniform [Head, Head, Head, Head, Tail]
Head | 0.8000
Tail | 0.2000
```
The `Show` instance of `Dist` ensures that it sums up the probabilities of the same outcome, giving the above. We don't show the `Show` instance here, that and other useful probabilistic combinators are 
presented in this [blog post](https://dennybritz.com/posts/probability-monads-from-scratch/).

#### Monad instance 

The most important bit for modeling Monty Hall is the Monad instance of `Dist`. We will use the Monad instance to model conditional discrete distribution. Given two discrete random variables `X` and `Y`,
we can compute their joint distribution (given that they are not mutually independent) as follows:

$$ Pr({X = x} \cap {Y = y}) = Pr(X = x | Y = y) . Pr (Y = y)$$

We can convert this idea to the Monad instance quite naturally:

```Haskell
instance Monad Dist where
  (Dist xs) >>= f = Dist $ do
    (x, p)  <- xs
    (y, p') <- unpackDist (f x)
    return (y, p * p')
```
The `p * p'` is multiplying the probabilities of the two outcomes. The `join` function of the Monad instance can allow us consider an alternate view of the above. Considering a *distribution of distributions* (please don't try to visualise),
we want to flatten it into a single distribution as follows:

```Haskell
join :: Dist (Dist a) -> Dist a
join (Dist dist) = Dist $ do -- dist  :: [(Dist a, Prob)]
  (Dist dista, prob) <- dist -- dista :: [(a, Prob)]
  (a, prob2)         <- dista
  return (a, prob * prob2)
```

#### Monty Hall

Armed with this very small set of combinators above, we now model the Monty Hall problem. We start by defining the outcome type:

```Haskell
data Outcome = Win | Loss deriving (Show, Eq, Ord)
```

And now, lets intuitively specify the switching strategy : *If our first choice is the winning door*, followed by which Monty Hall opens a door that contains a goat, 
*if we switch, then we will certainly lose*. That is because we already selected the winning door, and the switch will cost us the victory. However, *if our first choice is the losing door*,
followed by which Monty Hall opens the door containing a goat, *if we switch, then we certainly win*. Because, we chose the losing door, Monty Hall opened the other losing door and the
remaining door certainly assures our victory.

In the specification above we used the language of *certain victory* or *certain loss*, so we need to model certainty. And that is simply modeled as:

```Haskell
certainly :: a -> Dist a
certainly a = Dist [(a, 1.0)]
```

So, basically one outcome that has the probability of 1.0. And, with that we can easily translate the English specification above, to the Haskell snippet below:

```Haskell
switching :: Dist Outcome
switching = do
  firstChoice <- uniform [Win,Loss,Loss]
  if (firstChoice == Win)
  then {- switching will -} certainly Loss
  else {- switching will -} certainly Win
```

The comment `{- switching will -}` is simply added such that the specification almost translates to *literal* English. Now, we can observe the distributions:

```Haskell
> switching
 Win | 0.6667
Loss | 0.3333
```

This shows us that switching our choice will allow us to win 66% of times, while losing 33% of times. Given the first choice has the following distributions:

```Haskell
> uniform [Win, Loss, Loss]
 Win | 0.3333
Loss | 0.3333
Loss | 0.3333
```

Our computation proceeds by certainly losing if we switch after winning the first round, while certainly winning if we switch after losing the first round. Hence the calculation proceeds as:

```Haskell
 Win ~> certainly Loss| 0.3333 * 1.0
Loss ~> certainly Win | 0.3333 * 1.0
Loss ~> certainly Win | 0.3333 * 1.0

==>

Loss | 0.3333
Win  | 0.3333
Win  | 0.3333

==>

 Win | 0.6667
Loss | 0.3333
```

Hence, the contestant should switch!

Source code for the above available at : [Abhiroop/bayes](https://github.com/Abhiroop/bayes)

