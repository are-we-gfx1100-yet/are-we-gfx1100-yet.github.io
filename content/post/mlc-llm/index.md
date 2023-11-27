---
title: Use MLC LLM for Windows on RX 7900 XTX
date: 2023-11-26
tags:
- llm
- windows
- tutorial
---

Hi, friends, long time no see. I haven't updated this site for a while.

The main reason is that I often need to work on Windows, and the restart time of my ASUS motherboard is particularly long, so I gradually became lazy to boot into Linux.

The secondary reason is that there hasn't been much worth updating lately. Although ROCm 5.7 has been released, it doesn't really affect the content of this site. Personally, I recommend using ROCm 5.6, which is the version supported by the stable PyTorch at the moment.

Recently, the article about AITemplate for RX 7900 XTX was finally published. Although the related branches have been inactive for about half a year, it is still worth mentioning. Hopefully, there may be a ready-to-use Flash Attention implementation for RX 7900 XTX.

I have heard about some interesting LLMs recently, but due to my laziness to boot into Linux, I thought about MLC LLM, which was mentioned before. It is a LLM implementation that can max out the performance of RX 7900 XTX. The use of the TVM Unity architecture is also very interesting, but I haven't tried it in the past, due to its incomplete ecosystem. Nowadays, this project has matured a lot, so I decided to give it a try.

This practice was being carried out on Windows 10 22H2 and Ubuntu 22.04 for WSL2, without the need to boot into Linux.

## What is MLC LLM?

It's a machine learning architecture that converts parameters, compiles models and executes them on different platforms, in a highly optimized way. In their articles, the execution performance of RX 7900 XTX can come close to that of RTX 3090 Ti.

I believe its approach is similar to AITemplate and TensorRT, with comparable performance and similar pros and cons, but it supports many platforms and has the best support for AMD GPUs.

Some related Python packages here:

* `mlc-ai-nightly`
  * the TVM Unity compiler, required by both `mlc-chat-nightly` and `mlc-llm`
* `mlc-chat-nightly`
  * the MLC LLM Python package, used to inference on Windows
* `mlc-llm`
  * the MLC LLM package, `mlc-llm.build` is used to convert models

## Does it work?

First of all, install Git and Python 3.10 on your Windows, and make sure they are in the PATH and you can call them from the terminal. I am using Windows PowerShell here.

```powershell
cd to\your\workspace

python -m venv venv
.\venv\Scripts\activate
# for convenience
cp .\venv\Scripts\python.exe .\venv\Scripts\python3.exe

python3 -m pip install --pre -U -f https://mlc.ai/wheels mlc-ai-nightly mlc-chat-nightly

# https://llm.mlc.ai/docs/index.html#get-started

git lfs install
mkdir -p .\dist\prebuilt
git clone https://huggingface.co/mlc-ai/mlc-chat-Llama-2-7b-chat-hf-q4f16_1 .\dist\prebuilt\mlc-chat-Llama-2-7b-chat-hf-q4f16_1
git clone https://github.com/mlc-ai/binary-mlc-llm-libs .\dist\prebuilt\lib

# the last version of gradio 3
python3 -m pip install gradio==3.50.2

# start gradio for mlc chat and you can access it at http://127.0.0.1:7860
python3 -m mlc_chat.gradio --artifact-path .\dist\prebuilt
```

On my Windows 10 22H2 + Vulkan backend, it gives:

```Stats: prefill: 56.5 tok/s, decode: 101.4 tok/s```

for the first 256 tokens, which is already better than running 7B with ExLlama (v1) on RX 7900 XTX.

However, there are plenty of models that aren't prebuilt, and you need to build them yourself if you want to run them in MLC.

As of November 26, 2023, there is no instruction for building custom models directly on Windows, so WSL2 comes to the rescue.

## Prerequisites

* Git for Windows
* Python 3.10 on Windows, from official website
* Ubuntu 22.04 on WSL2, from Windows Store
  * Ubuntu 20.04 is not used because it doesn't include `lavapipe` out of the box

On Windows, Vulkan backend will be used, because unfortunately ROCm for Windows doesn't work with AI now.

## Build in WSL2 Ubuntu

```bash
sudo apt update
sudo apt install -y python3-pip python3-venv cmake vulkan-tools

cd workspace

python3 -m venv venv
source venv/bin/activate

python3 -m pip install --pre -U -f https://mlc.ai/wheels mlc-ai-nightly

git clone https://github.com/mlc-ai/mlc-llm --recursive && pushd mlc-llm && python3 -m pip install . && popd

# although it's built for ubuntu 20.04, it works in ubuntu 22.04 too
curl -LO https://github.com/mstorsjo/llvm-mingw/releases/download/20231114/llvm-mingw-20231114-ucrt-ubuntu-20.04-x86_64.tar.xz
tar -xf llvm-mingw-20231114-ucrt-ubuntu-20.04-x86_64.tar.xz

export MODEL_DIR=path/to/your/model/dir
export LLVM_MINGW_DIR=llvm-mingw-20231114-ucrt-ubuntu-20.04-x86_64
# the most tricky part here, as wsl doesn't really support vulkan
# a software renderer running on cpu is needed here
# lavapipe works pretty well in this case, and swiftshader does not
export VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/lvp_icd.x86_64.json

# append --use-safetensors if the original model is stored in .safetensors
# to avoid ValueError: Multiple weight shard files without json map is not supported
python3 -m mlc_llm.build --model "$MODEL_DIR" --target vulkan --quantization q4f16_1 --llvm-mingw "$LLVM_MINGW_DIR"

# built artifacts are located at ./dist
# $MODEL_NAME is basically the basename of $MODEL_DIR
# q4f16_1 may vary if you use another --quantization option
du -ah dist
# in dist/$MODEL_NAME-q4f16_1, you can find a params directory, and a $MODEL_NAME-vulkan.dll:
# * the params directory is the converted parameters
# * the $MODEL_NAME-vulkan.dll directory is the compiled model running in vulkan

# just move the dist directory to anywhere outside of wsl
mv dist /mnt/drive/to/your/workspace
```

## Run in Windows 10

Assuming you have followed the "Does it work?" section, the steps are simple (in workspace directory):

* Copy `dist/$MODEL_NAME-q4f16_1/params` to `dist/prebuilt/$MODEL_NAME-q4f16_1`
* Copy `dist/$MODEL_NAME-q4f16_1/$MODEL_NAME-q4f16_1-vulkan.dll` to `dist/prebuilt/lib`
* Launch WebUI just like what we have done in "Does it work?"
  * `python3 -m mlc_chat.gradio --artifact-path .\dist\prebuilt`
  * Open `http://127.0.0.1:7860` in browser
  * Enjoy!

## Caveats

### It works but generates bad results

You might need to open `dist/prebuilt/$MODEL_NAME-q4f16_1/params/mlc-chat-config.json` and edit the `conv_template` property.

You can follow the instructions here:

* https://llm.mlc.ai/docs/get_started/mlc_chat_config.html#conversation-structure

The supported conversation templates are limited, and I didn't find one for Alpaca, and had to use `dolly` for testing, which might not work for some models.
