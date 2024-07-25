---
layout: post
title: Solving Monty Hall with the List monad (or Probabilistic Programming)
---

Continuing from my [Bayes Theorem-based blog post](https://abhiroop.github.io/Monty-Hall/) on solving the Monty-Hall problem, I will now show a rather elegant way to model the solution that exploits
the humble List monad. To remind the reader, the Monty Hall problem involves a contestant initially choosing one of three doors, hoping to find a car behind it. Monty Hall then opens a different door, revealing a goat, and asks the contestant *if they want to switch their initial choice*. The goal of the solution is to guide the contestants to make this switching choice such that they have a higher probability of winning. This post was inspired by a [less-than 100 lines implementation of a probabilistic programming monad](https://dennybritz.com/posts/probability-monads-from-scratch/) in Haskell 

We will begin by representing discrete distributions as a Haskell datatype:

```haskell
type Prob = Double -- probability of an outcome

newtype Dist a = Dist { unpackDist :: [(a, Prob)] }
```

This is a nicer reformulation of distributions that the List monad could naturally model. For example, in an experiment with 5 tosses (modelled as `data Toss = Head | Tail`), that leads to 
4 Heads and 1 Tail, our `Dist` type will model this as `Dist [(Head, 0.8), (Tail, 0.2)]`. However, we could have modelled this as a plain list as `[Head, Head, Head, Head, Tail]`. It is simply that the `Dist` type is nicer to work with (but it is a wrapper on the plain `List`).

Next, to compute probabilities accurately, it is easier for our representation if we normalise the distribution. We can do that using:

```haskell
normP :: [(a, Prob)] -> [(a, Prob)]
normP xs = map (\(x, y) -> (x, y / (sumP xs))) xs
  where
    sumP :: [(a, Prob)] -> Prob
    sumP = sum . map snd
```

Given the above, we can now define the uniform distribution:

```haskell
uniform :: [a] -> Dist a
uniform = Dist . normP . map (, 1.0)

> uniform [Head, Head, Head, Head, Tail]
Head | 0.8000
Tail | 0.2000
```
The `Show` instance of `Dist` ensures that it sums up the probabilities of the same outcome, giving the above. We don't show the `Show` instance here, that and other useful probabilistic combinators are 
presented in this [blog post](https://dennybritz.com/posts/probability-monads-from-scratch/).

#### Monad instance 

The Monad instance of `Dist` is the most important construct for modelling Monty Hall. We will use the Monad instance to model conditional discrete distribution. Given two discrete random variables `X` and `Y`,
we can compute their joint distribution (given that they are not mutually independent) as follows:

$$ Pr({X = x} \cap {Y = y}) = Pr(X = x | Y = y) . Pr (Y = y)$$

We can convert this idea to the Monad instance quite naturally:

```haskell
instance Monad Dist where
  (Dist xs) >>= f = Dist $ do
    (x, p)  <- xs
    (y, p') <- unpackDist (f x)
    return (y, p * p')
```
The `p * p'` is multiplying the probabilities of the two outcomes. The `join` function of the Monad instance can allow us to consider an alternate view of the above. Considering a *distribution of distributions* (please don't try to visualise),
we want to flatten it into a single distribution as follows:

```haskell
join :: Dist (Dist a) -> Dist a
join (Dist dist) = Dist $ do -- dist  :: [(Dist a, Prob)]
  (Dist dista, prob) <- dist -- dista :: [(a, Prob)]
  (a, prob2)         <- dista
  return (a, prob * prob2)
```

#### Monty Hall

Armed with this small set of combinators above, we now model the Monty Hall problem. We start by defining the outcome type:

```haskell
data Outcome = Win | Loss deriving (Show, Eq, Ord)
```

And now, let's intuitively specify the switching strategy : *If our first choice is the winning door*, followed by which Monty Hall opens a door that contains a goat, 
*if we switch, then we will certainly lose*. That is because we already selected the winning door, and the switch will cost us the victory. However, *if our first choice is the losing door*,
followed by Monty Hall opening the door containing a goat, *if we switch, then we certainly win*. Because we chose the losing door, Monty Hall opened the other losing door and the
remaining door certainly assures our victory.

In the specification above we used the language of *certain victory* or *certain loss*, so we need to model certainty. And that is modelled as:

```haskell
certainly :: a -> Dist a
certainly a = Dist [(a, 1.0)]
```

So, one outcome that has a probability of 1.0. And, with that, we can easily translate the English specification above, to the Haskell snippet below:

```haskell
switching :: Dist Outcome
switching = do
  firstChoice <- uniform [Win,Loss,Loss]
  if (firstChoice == Win)
  then {- switching will -} certainly Loss
  else {- switching will -} certainly Win
```

The comment `{- switching will -}` is added such that the specification almost translates to *literal* English. Now, we can observe the distributions:

```haskell
> switching
 Win | 0.6667
Loss | 0.3333
```

This shows us that switching our choice will allow us to win 66% of the time while losing 33% of the time. The first choice has the following distributions:

```haskell
> uniform [Win, Loss, Loss]
 Win | 0.3333
Loss | 0.3333
Loss | 0.3333
```

Our computation proceeds by *certainly* losing if we switch after winning the first round, while *certainly* winning if we switch after losing the first round. Hence the calculation proceeds as:

```haskell
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

Source code for the above is available at [Abhiroop/bayes](https://github.com/Abhiroop/bayes)

