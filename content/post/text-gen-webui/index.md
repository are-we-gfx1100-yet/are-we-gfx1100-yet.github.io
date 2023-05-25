---
title: Run Text Generation WebUI on Ubuntu 22.04 with RX 7900 XTX
date: 2023-05-24
tags:
- torch
- text-gen-webui
- quantization
- bitsandbytes
- gptq
- tutorial
---

## Prerequisites

### [Install AMDGPU driver with ROCm](https://docs.amd.com/bundle/ROCm-Installation-Guide-v5.5/page/How_to_Install_ROCm.html)

### Download the following prebuilt wheels into `~/Downloads`

* `torch`
  * https://github.com/evshiron/rocm_lab/releases/download/v1.14.514/torch-2.0.1+gite19229c-cp310-cp310-linux_x86_64.whl
* `bitsandbytes`
  * https://github.com/evshiron/rocm_lab/releases/download/v1.14.514/bitsandbytes-0.37.2-py3-none-any.whl

If somehow the above links become outdated, you can always find latest links here:

* https://github.com/evshiron/rocm_lab/releases

and don't forget to use their new filenames in "Install" section.

## Install

```bash
# clone text-generation-webui
git clone https://github.com/oobabooga/text-generation-webui
pushd text-generation-webui

# activate venv
python3 -m venv venv
source venv/bin/activate

# install GPTQ-for-LLaMa-ROCm

mkdir -p repositories
pushd repositories

git clone https://github.com/WapaMario63/GPTQ-for-LLaMa-ROCm GPTQ-for-LLaMa

pushd GPTQ-for-LLaMa && python3 setup_rocm.py install && popd

popd

# install dependencies and uninstall bitsandbytes
pip install -r requirements.txt
pip uninstall -y bitsandbytes

# install custom torch and bitsandbytes
pip install ~/Downloads/torch-2.0.1+gite19229c-cp310-cp310-linux_x86_64.whl
pip install ~/Downloads/bitsandbytes-0.37.2-py3-none-any.whl
```

## Launch

```bash
source venv/bin/activate

python3 ./server.py --listen
```

## Notes

4-bit BitsAndBytes inference doesn't work.

GPTQ quantization doesn't work.

## Caveats

### `RuntimeError: HIP error: invalid argument`

This error usually occurs when the GPU architecture detection malfuntions. Let's specify it explicitly:

```bash
export HSA_OVERRIDE_GFX_VERSION=11.0.0

python3 ./server.py --listen
```

## Performance

Tested on RX 7900 XTX using PCI-E 3.0 x16 slot.

### Transformers inference with 13.6GB VRAM

```
INFO:Loading 7B-hf...
INFO:Loaded the model in 6.34 seconds.
Output generated in 9.37 seconds (21.24 tokens/s, 199 tokens, context 27, seed 1005554483)
```

### 8-bit BitsAndBytes inference with 7.9GB VRAM

```
INFO:Loading 7B-hf...
INFO:Loaded the model in 6.65 seconds.
Output generated in 40.70 seconds (4.89 tokens/s, 199 tokens, context 6, seed 603994963)
```

### 4-bit GPTQ inference with 4.5GB VRAM

```
INFO:Loading Wizard-Vicuna-7B-Uncensored-GPTQ...
INFO:Loaded the model in 2.10 seconds.
Output generated in 17.17 seconds (17.42 tokens/s, 299 tokens, context 52, seed 722332956)
```

## Conclusions

It is exciting to see that 4-bit GPTQ can do inference smoothly on RX 7900 XTX, which demonstrates that ROCm platform has a reliable 4-bit quantization solution.

Although 8-bit BitsAndBytes can also do inference, its performance is significantly degraded, let's look forward to its future improvements.
