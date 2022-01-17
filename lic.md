### Title : Functional Programming for Embedded Systems

#### Abstract

Embedded Systems application development has been traditionally carried out in low-level machine-oriented programming languages like C or Assembler that can result in unsafe, error-prone and difficult-to-maintain code. Functional programming with features such as higher-order functions, algebraic data types, polymorphism, strong static typing and automatic memory management seems to be a suitable candidate to address the issues with low-level languages and simplifying embedded-systems programming.

However, embedded systems usually run on heavily memory-constrained devices with memory in the order of hundreds of kilobytes and applications running on such devices embody the general characteristics of being (i) I/O-bound, (ii) concurrent and (iii) timing-aware.
Popular functional language compilers and runtimes either do not fare well with such scarce memory resources or do not provide high-level abstractions that address all the three listed characteristics.


This work fills this gap by investigating and proposing high-level abstractions specialised for I/O-bound, concurrent and timing-aware embedded-systems programs. We implement the proposed abstractions on eagerly-evaluated, statically-typed functional languages running natively on microcontrollers. Our contributions are divided into two parts -

Part 1 presents a functional reactive programming language - Hailstorm - that tracks side effects like I/O in its type system using a feature called *resource types*. The language uses a combination of resource types and Arrowized Functional Reactive Programming (AFRP) operators to write embedded systems programs running on the GRiSP microcontroller board.

Part 2 comprises two papers that describe the design and implementation of a virtual machine (VM) - SenseVM - that can host functional languages while providing support for message-passing based concurrency (inspired from ConcurrentML). It also supports a message-passing based IO interface that translates between low-level interrupt based and memory-mapped peripherals and includes a special operator to express timing behaviours. SenseVM currently hosts a Caml-like functional language and is capable of hosting DSLs like Hailstorm as well. Our evaluation of the SenseVM is carried out on the popular STM32F4 and the NRF52 microcontroller boards. We discuss the interpreter and garbage collection overhead on the response-time and also measure precision and jitter of timed operations on SenseVM.

#### Introduction

[TODO]

#### Papers

##### Part 1. A native language.

- [Hailstorm : A Statically-Typed, Purely Functional Language for IoT Applications](https://abhiroop.github.io/pubs/hailstorm/) - PPDP 2020 - [DOI](https://dl.acm.org/doi/10.1145/3414080.3414092)

##### Part 2. A virtual machine

- [Higher-Order Concurrency for Microcontrollers](https://abhiroop.github.io/pubs/sensevm_mplr) - MPLR 2021 - [DOI](https://dl.acm.org/doi/10.1145/3475738.3480716)
- [Programming Embedded Systems with SenseVM](https://raw.githubusercontent.com/Abhiroop/Abhiroop.github.io/master/pubs/SenseVM_ECOOP.pdf) (Under review)
