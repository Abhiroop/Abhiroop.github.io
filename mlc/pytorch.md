---
layout: page
title: PyTorch
permalink: /mlc/pytorch
---

All development is being carried out in this repo: [https://github.com/Abhiroop/mlcomp](https://github.com/Abhiroop/mlcomp)

The first overwhelming thing in the ML compiler world is the galore of software versions. This is PyTorch, the frontend and we will already encounter a number of moving parts, where we will have to pin certain version numbers to follow along. Firstly, the bread and butter of ML - Python. We will go with Python version 3.11. Assuming that you have pinned down this version number, we will create a virtual environment for this version of Python and correspondingly I will install several packages, with their individual version numbers pinned to this particular Python:

```
python3.11 -m venv mlcomp
source mlcomp/bin/activate

# Download the matching wheels
wget https://github.com/llvm/torch-mlir/releases/download/snapshot-20240102.1071/torch-2.2.0.dev20231204+cpu-cp311-cp311-linux_x86_64.whl
wget https://github.com/llvm/torch-mlir/releases/download/snapshot-20240102.1071/torch_mlir-20240102.1071-cp311-cp311-linux_x86_64.whl

# Install both wheels
pip install torch-2.2.0.dev20231204+cpu-cp311-cp311-linux_x86_64.whl torch_mlir-20240102.1071-cp311-cp311-linux_x86_64.whl


pip install matplotlib

# Verify
python -c "import torch, torch_mlir; print('OK')"
```

We have installed `torch`, `torch-mlir`, `matplotlib`, while ensuring version compatibility above. If all goes well you should get 'OK' above.

### Tensors

Even if you are distantly connected to ML compilers, you would have heard of the word "tensor" in some shape or form. Maybe as "Tensor Processing Units", a new architectural component found in modern mobiles (especially those manufactured by Google) or from the library Tensorflow.

A Tensor is a mathematical object that generalises scalars, vectors and matrices to higher dimensions. A Tensor defines something called a rank, which is the dimension or number of indices required to specify components of a tensor. For eg:

- a rank 0 tensor is a scalar
- a rank 1 tensor is a vector
- a rank 2 tensor is a matrix
- rank 3 and above a re higher-dimensional arrays

Some tensor ops in PyTorch (you can explore the entire library in your own time, which is itself a massive undertaking):

```python
import torch

## initialising a rank 2 tensor
data = [[1, 2], [3, 4]]
x_data = torch.tensor(data)

tensor = torch.rand(3, 4) ## 2 dimensional tensor so as to not explode your brains with higher dimensions

print(f"Shape of tensor: {tensor.shape}")
print(f"Datatype of tensor: {tensor.dtype}")
print(f"Device tensor is stored on: {tensor.device}")

## Tensor Multiplication
print(f"tensor * tensor \n {tensor * tensor}")

## Matrix Multiplication as tensors
print(f"tensor @ tensor.T \n {tensor @ tensor.T}")
```
