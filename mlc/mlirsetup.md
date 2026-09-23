---
layout: page
title: MLIR
permalink: /mlc/mlirsetup
---

All development is being carried out in this repo: [https://github.com/Abhiroop/mlcomp](https://github.com/Abhiroop/mlcomp)

In the last post we used `torch-mlir` to emit MLIR code fragment and now we will use MLIR (the defacto choice of machine learning compilers) to optimise our computational graph. But first, lets not take lightly the hassle of setting up MLIR. The MLIR project lives within the LLVM main repo, which as of this writing sits on version 23. For this write up I will use LLVM 21.

Now, I installed mlir/llvm 21 from the apt repositories. But, it will also be useful to run through some of the [Toy language tutorial](https://mlir.llvm.org/docs/Tutorials/Toy/) examples from the MLIR repo, to familiarise ourselves with MLIR concepts. For that, we need to check out the `llvm` main repo. Inside `llvm`, you should find the `mlir` repo with the following structure:

```
~/cpp/llvm-project/mlir (main) $ tree -L 1
.
├── benchmark
├── cmake
├── CMakeLists.txt
├── docs
├── examples
├── include
├── lib
├── LICENSE.TXT
├── Maintainers.md
├── python
├── README.md
├── test
├── tools
├── unittests
└── utils
```

Under `examples/toy` modify the `CMakeLists.txt` to the following:

```
cmake_minimum_required(VERSION 3.22)
project(Toy)

set(LLVM_CMAKE_DIR "/usr/lib/llvm-21/lib/cmake/llvm")
set(MLIR_CMAKE_DIR "/usr/lib/llvm-21/lib/cmake/mlir")

list(APPEND CMAKE_MODULE_PATH "${LLVM_CMAKE_DIR}" "${MLIR_CMAKE_DIR}")

include(AddLLVM)
include(AddMLIR)

include_directories("/usr/lib/llvm-21/include")
link_directories("/usr/lib/llvm-21/lib")

add_custom_target(Toy)
set_target_properties(Toy PROPERTIES FOLDER "MLIR/Examples")

macro(add_toy_chapter name)
  add_dependencies(Toy ${name})
  add_llvm_example(${name} ${ARGN})
  # Add the Demangle library to resolve itaniumDemangle symbol
  target_link_libraries(${name} PRIVATE LLVMDemangle)
endmacro(add_toy_chapter name)

add_subdirectory(Ch1)
```

Then do the following:

```
mkdir build
cd build
cmake -G Ninja .. -DLLVM_DIR=/usr/lib/llvm-21/cmake -DMLIR_DIR=/usr/lib/llvm-21/cmake/mlir
ninja toyc-ch1
```

To verify we have everything working with the MLIR toy compiler, from the `mlir` base repo try this:

```
examples/toy/build/Ch1/toyc-ch1 test/Examples/Toy/Ch1/ast.toy -emit=ast
```

The above should emit:

```
  Module:
    Function 
      Proto 'multiply_transpose' @test/Examples/Toy/Ch1/ast.toy:4:1
      Params: [a, b]
      Block {
        Return
          BinOp: * @test/Examples/Toy/Ch1/ast.toy:5:25
            Call 'transpose' [ @test/Examples/Toy/Ch1/ast.toy:5:10
              var: a @test/Examples/Toy/Ch1/ast.toy:5:20
            ]
            Call 'transpose' [ @test/Examples/Toy/Ch1/ast.toy:5:25
              var: b @test/Examples/Toy/Ch1/ast.toy:5:35
            ]
      } // Block
    Function 
      Proto 'main' @test/Examples/Toy/Ch1/ast.toy:8:1
      Params: []
      Block {
        VarDecl a<> @test/Examples/Toy/Ch1/ast.toy:11:3
          Literal: <2, 3>[ <3>[ 1.000000e+00, 2.000000e+00, 3.000000e+00], <3>[ 4.000000e+00, 5.000000e+00, 6.000000e+00]] @test/Examples/Toy/Ch1/ast.toy:11:11
        VarDecl b<2, 3> @test/Examples/Toy/Ch1/ast.toy:15:3
          Literal: <6>[ 1.000000e+00, 2.000000e+00, 3.000000e+00, 4.000000e+00, 5.000000e+00, 6.000000e+00] @test/Examples/Toy/Ch1/ast.toy:15:17
        VarDecl c<> @test/Examples/Toy/Ch1/ast.toy:19:3
          Call 'multiply_transpose' [ @test/Examples/Toy/Ch1/ast.toy:19:11
            var: a @test/Examples/Toy/Ch1/ast.toy:19:30
            var: b @test/Examples/Toy/Ch1/ast.toy:19:33
          ]
        VarDecl d<> @test/Examples/Toy/Ch1/ast.toy:22:3
          Call 'multiply_transpose' [ @test/Examples/Toy/Ch1/ast.toy:22:11
            var: b @test/Examples/Toy/Ch1/ast.toy:22:30
            var: a @test/Examples/Toy/Ch1/ast.toy:22:33
          ]
        VarDecl e<> @test/Examples/Toy/Ch1/ast.toy:25:3
          Call 'multiply_transpose' [ @test/Examples/Toy/Ch1/ast.toy:25:11
            var: c @test/Examples/Toy/Ch1/ast.toy:25:30
            var: d @test/Examples/Toy/Ch1/ast.toy:25:33
          ]
        VarDecl f<> @test/Examples/Toy/Ch1/ast.toy:28:3
          Call 'multiply_transpose' [ @test/Examples/Toy/Ch1/ast.toy:28:11
            var: a @test/Examples/Toy/Ch1/ast.toy:28:30
            var: c @test/Examples/Toy/Ch1/ast.toy:28:33
          ]
      } // Block
```