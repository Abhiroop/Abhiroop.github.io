---
layout: page
title: Functional Programming for Embedded Systems
permalink: /lic/
---


#### Abstract

Embedded Systems application development has traditionally been carried out in low-level machine-oriented programming languages like C or Assembler that can result in unsafe, error-prone and difficult-to-maintain code. Functional programming with features such as higher-order functions, algebraic data types, polymorphism, strong static typing and automatic memory management appears to be an ideal candidate to address the issues with low-level languages plaguing embedded systems.

However, embedded systems usually run on heavily memory-constrained devices with memory in the order of hundreds of kilobytes and applications running on such devices embody the general characteristics of being (i) I/O-bound, (ii) concurrent and (iii) timing-aware. Popular functional language compilers and runtimes either do not fare well with such scarce memory resources or do not provide high-level abstractions that address all the three listed characteristics.

This work attempts to address this gap by investigating and proposing high-level abstractions specialised for I/O-bound, concurrent and timing-aware embedded-systems programs. We implement the proposed abstractions on eagerly-evaluated, statically-typed functional languages running natively on microcontrollers. Our contributions are divided into two parts -



Part 1 presents a functional reactive programming language - Hailstorm - that tracks side effects like I/O in its type system using a feature called resource types. Hailstorm's programming model is illustrated on the GRiSP microcontroller board.

Part 2 comprises two papers that describe the design and implementation of Synchron, a runtime API that provides a uniform message-passing framework for the handling of software messages as well as hardware interrupts. Additionally, the Synchron API supports a novel timing operator to capture the notion of time, common in embedded applications. The Synchron API is implemented as a virtual machine - SynchronVM - that is run on the NRF52 and STM32 microcontroller boards. We present programming examples that illustrate the concurrency, I/O and timing capabilities of the VM and provide various benchmarks on the response time, memory and power usage of SynchronVM.

#### Introduction

[Introduction (Kappa)](https://raw.githubusercontent.com/Abhiroop/Abhiroop.github.io/master/Abhiroop_Lic_Kappa.pdf)

#### Papers

##### Part 1. A Domain-specific Language.

- [Hailstorm : A Statically-Typed, Purely Functional Language for IoT Applications](https://abhiroop.github.io/pubs/hailstorm/) - PPDP 2020 - [DOI](https://dl.acm.org/doi/10.1145/3414080.3414092)

##### Part 2. A Virtual Machine.

- [Higher-Order Concurrency for Microcontrollers](https://abhiroop.github.io/pubs/sensevm_mplr) - MPLR 2021 - [DOI](https://dl.acm.org/doi/10.1145/3475738.3480716)
- [Synchron - An API for Embedded Systems](https://raw.githubusercontent.com/Abhiroop/Abhiroop.github.io/master/pubs/SenseVM_ECOOP.pdf) (Work-in-progress)

##### Artifacts

[SynchronVM Repo](https://github.com/svenssonjoel/Sense-VM)

[Artifacts under review](https://chalmersuniversity.app.box.com/s/g1r0jxra1lcd69u5455s3lwjfcx05re1)
