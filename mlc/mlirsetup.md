---
layout: page
title: MLIR
permalink: /mlc/mlirsetup
---

All development is being carried out in this repo: [https://github.com/Abhiroop/mlcomp](https://github.com/Abhiroop/mlcomp)

🚧 Work in Progress

--------------------------------------------------------------

CMakeLists.txt

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

```
cmake -G Ninja .. -DLLVM_DIR=/usr/lib/llvm-21/cmake -DMLIR_DIR=/usr/lib/llvm-21/cmake/mlir
```

```
ninja toyc-ch1
```
