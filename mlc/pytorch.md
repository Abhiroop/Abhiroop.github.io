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

### Neural Networks

With automatic differentiation out of the way, training neural networks in PyTorch is almost a replica of what we saw above. The only notable thing we will show in this snippet is defining the neural network.

A typical training procedure for a neural network is as follows:

1. Define the neural network that has some learnable parameters (or weights)
2. Iterate over a dataset of inputs
3. Process input through the network
4. Compute the loss (how far is the output from being correct)
5. Propagate gradients back into the network’s parameters
6. Update the weights of the network, typically using a simple update rule: 
    `weight = weight - learning_rate * gradient`

We define a small neural network below:

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class Net(nn.Module):

    def __init__(self):
        super().__init__()
        # 1 input image channel, 6 output channels, 5x5 square convolution
        # kernel
        self.conv1 = nn.Conv2d(1, 6, 5)
        self.conv2 = nn.Conv2d(6, 16, 5)
        # an affine operation: y = Wx + b
        self.fc1 = nn.Linear(16 * 5 * 5, 120)  # 5*5 from image dimension
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, input):
        # Convolution layer C1: 1 input image channel, 6 output channels,
        # 5x5 square convolution, it uses RELU activation function, and
        # outputs a Tensor with size (N, 6, 28, 28), where N is the size of the batch
        c1 = F.relu(self.conv1(input))
        # Subsampling layer S2: 2x2 grid, purely functional,
        # this layer does not have any parameter, and outputs a (N, 6, 14, 14) Tensor
        s2 = F.max_pool2d(c1, (2, 2))
        # Convolution layer C3: 6 input channels, 16 output channels,
        # 5x5 square convolution, it uses RELU activation function, and
        # outputs a (N, 16, 10, 10) Tensor
        c3 = F.relu(self.conv2(s2))
        # Subsampling layer S4: 2x2 grid, purely functional,
        # this layer does not have any parameter, and outputs a (N, 16, 5, 5) Tensor
        s4 = F.max_pool2d(c3, 2)
        # Flatten operation: purely functional, outputs a (N, 400) Tensor
        s4 = torch.flatten(s4, 1)
        # Fully connected layer F5: (N, 400) Tensor input,
        # and outputs a (N, 120) Tensor, it uses RELU activation function
        f5 = F.relu(self.fc1(s4))
        # Fully connected layer F6: (N, 120) Tensor input,
        # and outputs a (N, 84) Tensor, it uses RELU activation function
        f6 = F.relu(self.fc2(f5))
        # Fully connected layer OUTPUT: (N, 84) Tensor input, and
        # outputs a (N, 10) Tensor
        output = self.fc3(f6)
        return output


net = Net()
print(net)

```

This defines a classic small convolutional neural network (similar to LeNet) for image classification. Once again I will not delve into the details of neural network but make small notes on the code fragments. 

* __init__ sets up the layers:
  - conv1: conv layer, 1 input channel → 6 output channels, 5×5 kernel (learns 6 filters).
  - conv2: conv layer, 6 → 16 channels, 5×5 kernel.
  - fc1, fc2, fc3: fully connected (linear) layers: 400→120, 120→84, 84→10 (10 output classes).
* forward defines the data flow:
  - conv1 → ReLU → 2×2 max‑pool (image shrinks 32×32 → 28×28 → 14×14)
  - conv2 → ReLU → 2×2 max‑pool (14×14 → 10×10 → 5×5)
  - Flatten to vector of size 16×5×5 = 400
  - Pass through fc1 (120 units) with ReLU, fc2 (84 units) with ReLU, fc3 (10 outputs, no activation)

The output is a 10‑element tensor representing class scores. The model learns the convolution filters and linear weights via training.
