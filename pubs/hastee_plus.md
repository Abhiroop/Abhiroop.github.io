---
layout: page
title: HasTEE+ : Confidential Cloud Computing and Analytics with Haskell
permalink: /pubs/hastee+
---
**Abstract -** Confidential computing is a security paradigm that enables the protection of confidential code and data in a co-tenanted cloud deployment using specialized hardware isolation units called Trusted Execution Environments (TEEs). By integrating TEEs with a Remote Attestation protocol, confidential computing allows a third party to establish the integrity of an \textit{enclave} hosted within an untrusted cloud. However, TEE solutions, such as Intel SGX and ARM TrustZone, offer low-level C/C++-based toolchains that are susceptible to inherent memory safety vulnerabilities and lack language constructs to monitor explicit and implicit information-flow leaks. Moreover, the toolchains involve complex multi-project hierarchies and the deployment of hand-written attestation protocols for verifying *enclave* integrity.

We address the above with HasTEE+, a domain-specific language (DSL) embedded in Haskell that enables programming TEEs in a high-level language with strong type-safety. HasTEE+ assists in multi-tier cloud application development by (1) introducing a \textit{tierless} programming model for expressing distributed client-server interactions as a single program, (2) integrating a general remote-attestation architecture that removes the necessity to write application-specific cross-cutting attestation code, and (3) employing a dynamic information flow control mechanism to prevent explicit as well as implicit data leaks. We demonstrate the practicality of HasTEE+ through a case study on confidential data analytics, presenting a data-sharing pattern applicable to mutually distrustful participants and providing overall performance metrics.

[Submitted to ESORICS 2024](https://abhiroop.github.io/pubs/HasTEE_ESORICS_Sarkar_Russo.pdf)


