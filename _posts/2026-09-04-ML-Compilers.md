---
layout: page
title: Navigating the Fragmented World of Machine Learning Compilers
---

Machine Learning and AI have taken over the world, with LLMs diffusing into our software development toolchains, search engines, travel planners and almost every facet of our lives.
Being a compiler engineer, however, my curiosity has been taken over by how these so-called "models" are being deployed in software. Our traditional software compiler toolchains would involve
an average programming language containing arithmetic, functions, recursion/loops, and other control flow instructions and various optimisation passes to **lower** the high-level languages into
an assembly language like x86 or ARM instruction sets. During the 60s, this world was also incredibly fragmented, with several competing and non-standard instruction sets until Intel won.

Today, the world of machine learning compilers is, I believe, living in its own "1960s Wild West" era. It is incredibly fragmented, and the chaos arises from every part of the pipeline: 
frontend rivals like PyTorch, JAX, and TensorFlow, to high-level representations like ONNX, down to MLIR's dizzying array of dialects and the endless sea of hardware backends they are forced to target.
It is a nightmare for the compiler engineer to navigate this complex landscape. Before my PhD, I would be arrogant enough to say that I alone would be the one to unify this fragmented world into one uniform
standard compiler toolchain, but a PhD later, the [XKCD standards comic](https://xkcd.com/927/) is not lost on me. 

So I decided to pick up the more humble goal of helping the poor compiler engineer navigate this fragmented world with my own opinionated choices from each part of the pipeline and showing how to
take an ML model and compile it down to a GPU target. My choices are mostly informed by the most popular tool (say, PyTorch as a frontend) at the moment of writing, which in a fragmented world
keeps evolving (eg: JAX might make PyTorch irrelevant in 3 years), but at least that gives a starting point to my fellow compiler writers. I shall keep this post as the central post and, in the following
links, point to various parts that you can jump to read and understand. I hope you have fun, and excuse my mistakes, as I am myself learning the landscape with you.


1. The PyTorch Frontend
