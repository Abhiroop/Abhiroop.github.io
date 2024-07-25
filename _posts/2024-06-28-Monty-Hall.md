---
layout: post
title: The Monty Hall Problem and Bayes Theorem
---
Today, while reading Steven Pinker's latest book on critical thinking, *Rationality*, I was scribbling a solution to the Monty Hall problem. The Monty Hall problem is probably one of the most well-studied 
pop-science problem statement with different intuitive explanations of the solution. In case, dear reader, you have been living under a rock here is the problem statement:

**Setup**: A game show where you as a contestant have to guess to win a prize. The game comprises three doors. Behind one door is a car (the prize), and behind the other two doors are goats.

**Initial Choice**: You choose one of the three doors.

**Host's Action**: The host, Monty Hall, who knows what is behind each door, opens one of the remaining two doors, revealing a goat.
    
**Decision Point**: You are then allowed to stick with your original choice or switch to the other unopened door.

**Question: How should you maximise your chances of winning the car?**

Now, somehow through various coffee table discussions, I always knew the answer was that the contestant should switch their choice. So many times I have had this encounter that I realised I never bothered to 
calculate or question my intuition on why the contestant should switch. However, the book informed me that even some of the leading mathematicians and statisticians of the era, including the legendary 
[Paul Erdős](https://en.wikipedia.org/wiki/Paul_Erd%C5%91s), were stumped by the answer to the problem. Erdős was convinced of the solution only after he witnessed simulations rather than a proof or calculation!

So, I decided to reuse my ninth-standard probability lessons of Bayes Theorem to craft a solution. I have not at this point read through any solution, except the one that Pinker provides in his book.
Pinker's solution (I am sure this particular solution was not Pinker's proposal but his explanation of a known solution) is quite different and uses a gameplay simulation and informal intuitions to drive home the result (and I think his explanation works). The Goodreads reviews particularly point out that and 
appreciate Pinker's lucid and intuitive explanation. If you are interested in reading more about his solution, I encourage you to purchase the [book.](https://stevenpinker.com/publications/rationality-what-it-why-it-seems-so-scarce-and-why-it-matters)
However, I will use an established result of the Bayes theorem of conditional probability, which states:

$$ Pr(A|B) =  \frac{Pr(B|A) . Pr (A)}{Pr(B)} $$


I have written enough PL papers that I am itching to start explaining the notation and domains, but it is 3:25 am and I am hoping that the reader can follow along with a basic ninth standard notation. If not, please jump to the wiki article on [Bayes Theorem](https://en.wikipedia.org/wiki/Bayes%27_theorem).

Now, let's consider the Monty Hall problem and assume that among the three doors, the contestant chooses the first one. Seeing this, Monty Hall opens the third door to reveal a goat. Then, Monty Hall asks 
the contestant if they want to switch. Let me draw the scenario at this point

```
   ---         ---         ---
  | C |       |   |       | M |
  Door 1      Door 2      Door 3
```

The `C` on Door 1 marks that the contestant chooses the door and the `M` on Door 3 marks that Monty Hall opened it to reveal a goat. 
The contestant now needs to maximise their probability of winning a car. Should they stick with Door 1 or switch to Door 2? In effect, they need to calculate:

$$ Pr(car\ in\ 2|my\ choice\ 1,\ monty\ opened\ 3) =  \frac{Pr(my\ choice\ 1,\ monty\ opened\ 3|car\ in\ 2) . Pr (car\ in\ 2)}{Pr(my\ choice\ 1,\ monty\ opened\ 3)} $$

Let's calculate the probabilities on the right side. `Pr(car in 2)` is the simplest one. Given that it is not constrained by any conditions, the probability that the car is in one of the three doors (in this case door 2) is simply `1/3`

Next, `Pr(my choice 1, monty opened 3|car in 2)`. This one is interesting. Given that the car is in door 2, what is the probability that Monty Hall opened door 3, while the contestant chose door 1? This
one is interesting because the problem statement makes certain assumptions - (1) *Monty Hall actually knows where the car is*, (2) *Monty Hall wants the show to last at least two rounds*. Holding assumption 2, Monty Hall will intentionally open a door with a goat such that the show can progress to the next round. Given that, in our conditional probability scenario, Monty can never open door 2, because the car will be revealed (remember assumption 1 that Monty knows where the car is) and the show won't progress to the next round. Similarly, Monty Hall can never open door 1, because the contestant has already made door 1 their choice. If door 1 houses a goat or a car, in both cases assumption 2 is invalidated and the show abruptly ends.
Hence, the only choice that Monty Hall has is to open Door 3. So, the overall conditional probability `Pr(my choice 1, monty opened 3|car in 2)` is in fact `1`! Door 3 is the only choice Monty Hall has.

Finally, `Pr(my choice 1, monty opened 3)`. This is a weaker version of the previous probability, where the condition `car in 2` is no longer in place. So, now Monty Hall cannot open door 1 because the
contestant has already chosen door 1 and opening door 1 will abruptly end the show (as explained earlier). Now, the probability of Monty opening door 3 is 50% or `1/2` because the constraint of the car is gone, and
Monty Hall needs to choose 1 out of 2 doors (Doors 2 and 3).

With the above, we will have:

$$ Pr(car\ in\ 2|my\ choice\ 1,\ monty\ opened\ 3) =  \frac{Pr(my\ choice\ 1,\ monty\ opened\ 3|car\ in\ 2) . Pr (car\ in\ 2)}{Pr(my\ choice\ 1,\ monty\ opened\ 3)} $$

$$ Pr(car\ in\ 2|my\ choice\ 1,\ monty\ opened\ 3) =  \frac{1 . \frac{1}{3}}{\frac{1}{2}} $$

$$ Pr(car\ in\ 2|my\ choice\ 1,\ monty\ opened\ 3) =  \frac{2}{3} $$



Equivalently, we can calculate the probability of the car being behind Door 1 as simply `1 - 2/3 = 1/3`. Hence, **the contestant should switch**, as they have a 66.67% probability of the car being behind Door 2.

Given that Steven Pinker's book is about *thinking styles*, I realised that I attempt to approach most problems quite *mechanically*. I observed this pattern while solving the other puzzles in the book, where I typically try to abstract the problem into a logical/math domain and apply established/known results to answer the question, rather than using physical intuition.
Quiet often this style is described as *dry* and *procedural*, as opposed to Pinker's *lucid* and *intuitive* style, but there is sometimes merit in dryness.

> Mathematics is a game played according to certain simple rules with meaningless marks on paper - David Hilbert
