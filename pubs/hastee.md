---
layout: page
title: HasTEE - Confidential Computing on Trusted Execution Environments with Haskell
permalink: /pubs/hastee
---
**Abstract -** Trusted Execution Environments (TEEs) are hardware en-
forced memory isolation units, emerging as a pivotal security
solution for security-critical applications. TEEs, like Intel
SGX and ARM TrustZone, allow the isolation of confidential
code and data within an untrusted host environment, such as
the cloud and IoT. Despite strong security guarantees, TEE
adoption has been hindered by an awkward programming
model. This model requires manual application partitioning
and the use of error-prone, memory-unsafe, and potentially
information-leaking low-level C/C++ libraries.

We address the above with *HasTEE*, a domain-specific language (DSL) embedded in Haskell for programming TEE
applications. HasTEE includes a port of the GHC runtime
for the Intel-SGX TEE. HasTEE uses Haskell’s type system
to automatically partition an application and to enforce *Information Flow Control* on confidential data. The DSL, being
embedded in Haskell, allows for the usage of higher-order
functions, monads, and a restricted set of I/O operations to
write any standard Haskell application. Contrary to previous
work, HasTEE is lightweight, simple, and is provided as a
*simple security library*; thus avoiding any GHC modifications.
We show the applicability of HasTEE by implementing case
studies on federated learning, an encrypted password wallet,
and a differentially-private data clean room.

[Extended Paper accepted at Haskell Symposium 2023](https://abhiroop.github.io/pubs/HasTEE_SGX.pdf)

[Arxiv link](https://arxiv.org/abs/2307.13172)


