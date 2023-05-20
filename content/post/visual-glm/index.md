---
title: Run VisualGLM-6B on RX 7900 XTX
date: 2023-05-20
tags:
- torch
- torchvision
- glm
- visual
- tutorial
---

## Prerequisites

Download the following prebuilt wheels into `~/Downloads`.

* `torch`
  * https://github.com/evshiron/rocm_lab/releases/download/v1.14.514/torch-2.0.1+gite19229c-cp310-cp310-linux_x86_64.whl
* `torchvision`
  * https://github.com/evshiron/rocm_lab/releases/download/v1.14.514/torchvision-0.15.2+f5f4cad-cp310-cp310-linux_x86_64.whl

If somehow the above links become outdated, you can always find latest links here:

* https://github.com/evshiron/rocm_lab/releases

and don't forget to use their new filenames in "Install" section.

## Install

```bash
mkdir GLM
cd GLM

git clone https://github.com/THUDM/VisualGLM-6B

git clone --depth=1 https://huggingface.co/THUDM/visualglm-6b

# create default venv dir
python3 -m venv venv
source venv/bin/activate

# install custom torch and torchvision
pip install ~/Downloads/torch-2.0.1+gite19229c-cp310-cp310-linux_x86_64.whl
pip install ~/Downloads/torchvision-0.15.2+f5f4cad-cp310-cp310-linux_x86_64.whl

cd VisualGLM-6B

# remove torch* from requirements.txt
sed -i "/^torch.*$/d" requirements.txt

pip install -r requirements.txt

# use local model path
sed -i "s/THUDM\/visualglm-6b/..\/visualglm-6b/g" cli_demo_hf.py
```

## Launch

```bash
source venv/bin/activate

cd VisualGLM-6B

python3 cli_demo_hf.py
```

## Caveats

Do NOT use quantized models which require `cpm_kernels`. `cpm_kernels` is written in CUDA and hasn't been maintained for quite some time.

## ChatGLM?

This question is left as an after-school assignment for students to ponder.
