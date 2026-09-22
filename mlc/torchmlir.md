---
layout: page
title: torch-mlir
permalink: /mlc/torchmlir
---

All development is being carried out in this repo: [https://github.com/Abhiroop/mlcomp](https://github.com/Abhiroop/mlcomp)

We will now lower from a dead simple fully connected linear network, which will be one linear layer (4 inputs → 2 outputs) followed by ReLU. The tool that we will use for MLIR extraction is `torch-mlir`, which we have installed in the last part.

```python
import torch
import torch_mlir

class TinyModel(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = torch.nn.Linear(4, 2)
    def forward(self, x):
        return torch.relu(self.linear(x))

model = TinyModel()
example_input = torch.randn(1, 4)

# Pass the model and example input(s), not the exported program
mlir_module = torch_mlir.compile(
    model,
    example_input,               # or (example_input,)
    output_type=torch_mlir.OutputType.LINALG_ON_TENSORS
)

print(mlir_module)
```

For our initial draft, we will use this tiny model, which will simplify writing MLIR passes for the complete pipeline. As I build on this tutorial, I will show compilation of the full CNN that we saw in the last part. The most interesting part is the line `mlir_module = torch_mlir.compile.....`. The relevant parts are:

- `model`. The TinyModel being compiled. It must be a subclass of `torch.nn.Module`.
- `example_input`. A dummy input tensor with the same shape and dtype as real inputs. PyTorch uses it to trace the model: it runs the model once with this input to capture the computational graph. This graph is then converted to MLIR. You can also pass a tuple of inputs if the model takes multiple arguments.
- `output_type`. Tells `torch_mlir` which MLIR dialect to produce.
   * `OutputType.LINALG_ON_TENSORS` means the result should be in the Linalg dialect, operating on tensors (not buffers).
   * This is a high‑level, hardware‑agnostic representation that is easy to analyze and transform.
   * Other options exist (e.g., `TORCH`, `TOSA`, `STABLEHLO`), but we will continue with `LinAlg`.


Internally the following happens:

1. `torch_mlir.compile` uses `torch.export` to convert the model and example input into an `ExportedProgram`.
2. It lowers that `ExportedProgram` through a series of passes from PyTorch ops to MLIR’s `torch` dialect, then to `linalg`.
3. It returns an `mlir.ir.Module` object that we shall further process with MLIR tools.


The MLIR output from the above is the following:

```
#map = affine_map<(d0, d1) -> (d0, d1)>
#map1 = affine_map<(d0, d1) -> (d1, d0)>
#map2 = affine_map<(d0, d1) -> (0, d1)>
#map3 = affine_map<(d0, d1) -> (d1)>
module attributes {torch.debug_module_name = "TinyModel"} {
  ml_program.global private mutable @global_seed(dense<0> : tensor<i64>) : tensor<i64>
  func.func @forward(%arg0: tensor<1x4xf32>) -> tensor<1x2xf32> {
    %cst = arith.constant dense<[-0.257151306, -0.444812059]> : tensor<2xf32>
    %cst_0 = arith.constant dense<[[-0.466641545, 0.115759194, 0.138935328, -0.105179965], [-0.358416855, -0.238811493, 0.117786705, -0.0971438288]]> : tensor<2x4xf32>
    %cst_1 = arith.constant 0.000000e+00 : f32
    %0 = tensor.empty() : tensor<4x2xf32>
    %1 = linalg.generic {indexing_maps = [#map, #map1], iterator_types = ["parallel", "parallel"]} ins(%cst_0 : tensor<2x4xf32>) outs(%0 : tensor<4x2xf32>) {
    ^bb0(%in: f32, %out: f32):
      linalg.yield %in : f32
    } -> tensor<4x2xf32>
    %2 = tensor.empty() : tensor<1x2xf32>
    %3 = linalg.fill ins(%cst_1 : f32) outs(%2 : tensor<1x2xf32>) -> tensor<1x2xf32>
    %4 = linalg.matmul ins(%arg0, %1 : tensor<1x4xf32>, tensor<4x2xf32>) outs(%3 : tensor<1x2xf32>) -> tensor<1x2xf32>
    %5 = linalg.generic {indexing_maps = [#map2, #map3, #map], iterator_types = ["parallel", "parallel"]} ins(%4, %cst : tensor<1x2xf32>, tensor<2xf32>) outs(%2 : tensor<1x2xf32>) {
    ^bb0(%in: f32, %in_2: f32, %out: f32):
      %7 = arith.addf %in, %in_2 : f32
      linalg.yield %7 : f32
    } -> tensor<1x2xf32>
    %6 = linalg.generic {indexing_maps = [#map2, #map], iterator_types = ["parallel", "parallel"]} ins(%5 : tensor<1x2xf32>) outs(%2 : tensor<1x2xf32>) {
    ^bb0(%in: f32, %out: f32):
      %7 = arith.cmpf ugt, %in, %cst_1 : f32
      %8 = arith.select %7, %in, %cst_1 : f32
      linalg.yield %8 : f32
    } -> tensor<1x2xf32>
    return %6 : tensor<1x2xf32>
  }
}
```
