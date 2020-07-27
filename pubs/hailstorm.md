---
layout: page
permalink: /pubs/hailstorm
---

### Hailstorm : A Statically-Typed, Purely Functional Language forIoT Applications

**Abstract -** With the growing ubiquity of *Internet of Things* (IoT), more complex logic is being programmed on resource-constrained IoT devices, almost exclusively using the C 
programming language. While C provides low-level control over memory, it lacks a number of high-level programming abstractions such as higher-order functions, 
polymorphism, strong static typing, memory safety, and automatic memory management.

We present Hailstorm, a statically-typed, purely functional programming language that attempts to address the above problem. It is a high-level programming language 
with a strict typing discipline. It supports features like higher-order functions, tail-recursion and automatic memory management, to program IoT devices in 
a declarative manner. Applications running on these devices tend to be heavily dominated by I/O. Hailstorm tracks side effects like I/O in its type system using 
*resource types*. This choice allowed us to explore the design of a purely functional standalone language, in an area where it is more common to embed a functional core 
in an imperative shell. The language borrows the combinators of arrowized FRP, but has discrete-time semantics. The design of the full set of combinators is work in 
progress, driven by examples. So far, we have evaluated Hailstorm by writing standard examples from the literature (earthquake detection, a railway crossing system 
and various other clocked systems), and also running examples on the GRiSP embedded systems board, through generation of Erlang.

[Paper at PPDP 2020](https://abhiroop.github.io/pubs/hailstorm_ppdp.pdf)                   
[Extended Version](https://abhiroop.github.io/pubs/hailstorm.pdf)
