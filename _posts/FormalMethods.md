---
layout: post
title: A Whirlwind Tour of Formal Methods
---

This post attempts to take on the daunting task of providing a broad-strokes summary of the wide field of Formal Methods. At the outset, I should warn you that each subtopic I cover commands entire books of a thousand pages or more—and one could quite feasibly write a PhD thesis on a mere *sub-sub-subtopic*. Consequently, my summary must inevitably abstract away from many deeper issues that practising Formal Methods specialists might find highly relevant, and for that I apologise in advance.

The origins of Computer Science are often traced to Alan Turing’s landmark proof of the undecidability of the halting problem—a formalisation of Hilbert’s original [Entscheidungsproblem](https://en.wikipedia.org/wiki/Entscheidungsproblem). Turing showed that no general algorithm can decide, for an arbitrary program and input, whether the program will eventually terminate or run indefinitely. This result was later generalised in 1951 as [Rice's Theorem](https://en.wikipedia.org/wiki/Rice%27s_theorem), which states that not just program termination but in fact all non-trivial semantic properties of programs are undecidable.

Despite these foundational results, the discipline of Formal Methods exists to do precisely the opposite—to establish non-trivial semantic properties of programs and systems without executing them. It does so by employing program-specific heuristics (such as `loop invariants`) rather than attempting to devise a general algorithm for arbitrary programs, and by heavily over-approximating program behaviours while remaining within the boundaries of decidability. Many of these techniques date back to the earliest days of computer science, yet they have grown increasingly vital only over the past couple of decades as the IT industry finally begins to acknowledge the value of **correctness** alongside, or even above, sheer **performance**.

In this post, I classify formal methods into six broad categories and provide a general overview of the technique alongside links to successful industrial tools adopting the respective technique. As a bonus, I also discuss the so-called *lightweight formal methods*—techniques that are typically unsound but nonetheless deliver valuable results in practice.

Topics:

1. [Automata Theory and Reachability](#1-automata-theory-and-reachability)
2. Model Checking (Success stories - The Amazon paper)
3. Symbolic Execution and Constraint Solving
4. Axiomatic Basis for Programming (Program Logics - Hoare) (Success stories - not sure, used in places Dafny, maybe some OS verification by MSR)
5. Curry-Howard Correspondence (Proof Assistants) (Success stories - CompCert, sel4)
6. Abstract Interpretation (Success stories - Astree)
7. (Bonus) Lightweight Formal Methods

## 1. Automata Theory and Reachability



 (Reachability) (Success stories - ?)


Meta topics:
 Satisfiability, SMT solvers, etc

Lightweight FM: QuickCheck, Runtime Monitoring
