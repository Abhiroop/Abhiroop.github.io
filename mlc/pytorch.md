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
