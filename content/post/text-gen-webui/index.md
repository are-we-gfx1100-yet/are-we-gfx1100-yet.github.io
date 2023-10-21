---
title: Run Text Generation WebUI on RX 7900 XTX
date: 2023-05-24
weight: -50
tags:
- torch
- text-gen-webui
- quantization
- bitsandbytes
- gptq
- tutorial
---

## Prerequisites

* https://are-we-gfx1100-yet.github.io/post/a1111-webui/#prerequisites

Use `amdgpu-install --usecase=graphics,rocm` without `opencl`, which might cause HIP issues at the moment.

## Install

```bash
# clone text-generation-webui
git clone https://github.com/oobabooga/text-generation-webui
cd text-generation-webui

# activate venv
python3 -m venv venv
source venv/bin/activate

# install dependencies

pip3 install torch --index-url https://download.pytorch.org/whl/rocm5.6
# the wheels listed in requirements_amd.txt are built for this specified version of torch
# using other versions here will fail when loading dynamic libraries
pip3 install -r requirements_amd.txt
```

### BitsAndBytes

BitsAndBytes is used in `transformers` when `load_in_8bit` or `load_in_4bit` is enabled. Unfortunately it has bad ROCm support and low performance on Navi 31.

If you only want to run some LLMs locally, quantized models in GGML or GPTQ formats might suit your needs better.

To use BitsAndBytes for other purposes, a tutorial about building BitsAndBytes for ROCm with limited features might be added in the future.

Here is a promising fork if you are willing to try it by yourself (disclaimer: I haven't tested it yet):

* https://github.com/arlo-phoenix/bitsandbytes-rocm-5.6

## Launch

```bash
source venv/bin/activate

# use the first gpu if there are many
export HIP_VISIBLE_DEVICES=0
# override the gfx version
export HSA_OVERRIDE_GFX_VERSION=11.0.0

python3 ./server.py --listen
```

## Performance

Tested on RX 7900 XTX using PCI-E 4.0 x16 slot.

### 7B Transformers inference

```
INFO:Loading 7B-hf...
INFO:Loaded the model in 10.36 seconds.
Output generated in 11.64 seconds (28.08 tokens/s, 327 tokens, context 52, seed 832229644)
```

### 7B 4-bit AutoGPTQ inference

```
INFO:Loading Wizard-Vicuna-7B-Uncensored-GPTQ...
INFO:Loaded the model in 2.10 seconds.
Output generated in 8.93 seconds (45.90 tokens/s, 410 tokens, context 52, seed 159282383)
```

### 7B 4-bit ExLlama inference

```
INFO:Loading Wizard-Vicuna-7B-Uncensored-GPTQ...
INFO:Loaded the model in 1.44 seconds.
Output generated in 6.15 seconds (76.30 tokens/s, 469 tokens, context 51, seed 353336174)
```

## Caveats

### `RuntimeError: HIP error: invalid argument` or `Memory access fault`

These errors usually occur when the GPU is mistakenly recognized. Making it explicit should solve the problem:

```bash
# for navi 3x
export HSA_OVERRIDE_GFX_VERSION=11.0.0

# for navi 2x
export HSA_OVERRIDE_GFX_VERSION=10.3.0

python3 ./server.py --listen
```
