---
layout: page
title: Superword Level Parallelism in the Glasgow Haskell Compiler
permalink: /pubs/haskellvector
---

**Abstract -** Superword parallelism or vectorisation has been a popular high performance computing technique in supercomputers for nearly 50 years now.  With the rise of various 
memory intensive numerical computing as well as deep learning algorithms like neural networks,vectorisation is finding itself increasingly being used in commodity 
hardware as well.  For a long time imperative languages have ruled the rooster for high performance computing,but over the last decade the purely functional 
language Haskell has steadily caught up withthe speed and performance of languages like C in various benchmarks.  However vectorisation remains an unsolved problem 
for various Haskell compilers.  This thesis attempts to solve that issue by adding support for vectorisation to the Glasgow Haskell Compiler. It also presents a 
new library for vector programming called *lift-vector* which  provides a declarative API for vector programming. We show improvements in performance of certain 
numerical computing algorithms using the library and propose a way forward to automatically vectorise programs inside the Glasgow Haskell Compiler.

[Masters Thesis](https://abhiroop.github.io/pubs/Abhiroop_Masters_Thesis.pdf)
