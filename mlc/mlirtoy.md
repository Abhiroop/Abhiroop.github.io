---
layout: page
title: Intro to MLIR with the Toy example
permalink: /mlc/mlirtoy
---

All development is being carried out in this repo: [https://github.com/Abhiroop/mlcomp](https://github.com/Abhiroop/mlcomp)


🚧 Work in Progress

------------------------------------------------------------------


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