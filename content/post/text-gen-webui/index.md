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
pip3 install -r requirements.txt
# remove packages built for cuda
pip3 uninstall bitsandbytes gptq-for-llama auto-gptq exllama

# replace torch with the rocm one
pip3 uninstall torch
# pip3 install --pre torch --index-url https://download.pytorch.org/whl/nightly/rocm5.5
pip3 install --pre torch --index-url https://download.pytorch.org/whl/nightly/rocm5.6
```

### BitsAndBytes

BitsAndBytes is used in `transformers` when `load_in_8bit` or `load_in_4bit` is enabled. Unfortunately it has bad ROCm support and low performance on Navi 31.

If you only want to run some LLMs locally, quantized models in GGML or GPTQ formats might suit your needs better.

To use BitsAndBytes for other purposes, a tutorial about building BitsAndBytes for ROCm with limited features might be added in the future.

Here is a promising fork if you are willing to try it by yourself (disclaimer: I haven't tested it yet):

* https://github.com/arlo-phoenix/bitsandbytes-rocm-5.6

### GPTQ for LLaMA

GPTQ for LLaMA has now been superseded by AutoGPTQ. Use AutoGPTQ instead.

### AutoGPTQ

AutoGPTQ supports ROCm recently, and you can now do quantization with ROCm too.

Unfortunately, only builds for ROCm 5.4.2 are provided officially for now, so we have to build it ourselves.

```bash
cd text-generation-webui

source venv/bin/activate

mkdir repositories
cd repositories

git clone https://github.com/PanQiWei/AutoGPTQ
cd AutoGPTQ

ROCM_VERSION=5.6 pip3 install -e .
```

If it doesn't compile, edit [**/exllama/hip_compat.cuh](https://github.com/PanQiWei/AutoGPTQ/blob/v0.4.2/autogptq_cuda/exllama/hip_compat.cuh) like this:

> In `v0.4.1` and `v0.4.2`, the file is located at `autogptq_cuda/exllama/hip_compat.cuh`, and it's now located at `autogptq_extension/exllama/hip_compat.cuh` in the `main` branch. Many thanks to Xil for pointing out this change.

```cpp
#define rocblas_handle hipblasHandle_t
#define rocblas_operation_none HIPBLAS_OP_N
#define rocblas_get_stream hipblasGetStream
#define rocblas_set_stream hipblasSetStream
#define rocblas_hgemm __compat_hipblasHgemm

// EDIT: add this line here
#define hipblasHgemm __compat_hipblasHgemm
```

After that, run `ROCM_VERSION=5.6 pip3 install -e .` again.

### ExLlama

ExLlama has highly optimized kernels for GPTQ 4bit, which provides impressive performance for inference.

```bash
cd text-generation-webui

source venv/bin/activate

mkdir repositories
cd repositories

git clone https://github.com/turboderp/exllama
cd exllama

# there is no need to install exllama manually
# as exllama will build its extensions when used in text-generation-webui
# expect slow launch time for the first run
```

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

### `RuntimeError: HIP error: invalid argument`

This error usually occurs when the recognized GPU architecture is wrong. Let's specify it explicitly (for Navi 31):

```bash
export HSA_OVERRIDE_GFX_VERSION=11.0.0

python3 ./server.py --listen
```
