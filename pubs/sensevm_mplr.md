---
layout: page
title: Higher-Order Concurrency for Microcontrollers
permalink: /pubs/sensevm_mplr
---

**Abstract -** Programming microcontrollers involves low level interfacing with hardware and peripherals that are concurrent and reactive. Such programs are typically written in a mixture of C and assembly using concurrent language extensions (like FreeRTOS tasks and semaphores), resulting in unsafe, callback-driven, error-prone and difficult-to-maintain code.

We address this challenge by introducing SenseVM - a bytecode- interpreted virtual machine that provides a message passing based higher-order concurrency model, originally introduced by Reppy, for microcontroller programming. This model treats synchronous operations as first-class values (called Events) akin to the treatment of first-class functions in functional languages. This primarily allows the programmer to compose and tailor their own concurrency abstractions and additionally, abstracts away unsafe memory operations, common in shared-memory concurrency models, thereby making microcontroller programs safer, composable and easier-to-maintain.

Our VM is made portable via a low-level bridge interface, built atop the embedded OS - Zephyr. The bridge is implemented by all drivers and designed such that programming in response to a software message or a hardware interrupt remains uniform and indistinguishable. In this paper we demonstrate the features of our VM through an example, written in a Caml-like functional language, running on the nRF52840 and STM32F4 microcontrollers.

[Short Paper at MPLR 2021](https://abhiroop.github.io/pubs/sensevm_mplr.pdf)
