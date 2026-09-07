---
layout: page
title: PyTorch
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

