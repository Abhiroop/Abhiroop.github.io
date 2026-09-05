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
- rank 3 and above are higher-dimensional arrays

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

### torch.autograd

`torch.autograd` is PyTorch's automatic differentiation engine. Automatic Differentiation is a generalisation of the famous backpropagation algorithm that is used to calculate the derivative of the network error with respect to various neural network weights. This forms the foundation of neural network training. This algorithm is important enough to warrant its own space. I happened to give a [Papers We Love talk on automatic differentiation](https://abhiroop.github.io/slides/Automatic%20Differentiation.pdf). If you like video explanations head over to the [3Blue1Brown video](https://www.youtube.com/watch?v=tIeHLnjs5U8).

We will now walkthrough one step of gradient descent using `torch.autograd` on a sample model.

```python
import torch
import torch.nn as nn

# A minimal model substituting resnet
class TinyModel(nn.Module):
    def __init__(self):
        super().__init__()
        # Match the input shape (3*64*64 = 12288) and output 1000 classes
        self.fc = nn.Linear(3*64*64, 1000)

    def forward(self, x):
        # Flatten the image: (batch, 3, 64, 64) -> (batch, 3*64*64)
        x = x.view(x.size(0), -1)
        return self.fc(x)

model = TinyModel()
data = torch.rand(1, 3, 64, 64)
labels = torch.rand(1, 1000)

# Continue with the autograd example
prediction = model(data)
loss = (prediction - labels).sum()
loss.backward()

# Before gradient descent
print(model.fc.weight)

optim = torch.optim.SGD(model.parameters(), lr=1e-2, momentum=0.9)
optim.step() #gradient descent

# After gradient descent
print(model.fc.weight)
```

This will not be a full scale machine learning tutorial so I will explain the code above very briefly in terms of small notes:

1. `self.fc = nn.Linear(3*64*64, 1000)` creates a fully connected linear layer with 3 channels, height 64 and width 64. Output has 1000 classes - there are 1000 buckets in the famous ImageNet dataset (like coffee mug, car, etc) and the output will be a distribution among these classes, finally the one with the highest probability, will be chosen.
2. `forward` does a bunch of things but broadly flattens the image into a single long vector and then self.fc(x) applies the linear layer: it multiplies the flattened input by the weight matrix and adds the bias (essentially Wx + b from my autodiff slides).
3. `data` is the image to classify, `labels` is a ground-truth vector both randomly generated for example
4. `prediction = model(data)` first forward pass of AD.
5. `loss` name implies - calculates how far the forward pass is off from the ground truth.
6. `loss.backward()` is the reverse pass and stores the gradients in `model.fc.weight.grad`.
7. `optim.step` uses the gradients to update the weights of the model. 
