---
title: LoRA training with vladmandic/automatic on RX 7900 XTX
date: 2023-05-16
tags:
- torch
- torchvision
- automatic
- lora
- training
- tutorial
---

We will be using the training script from [vladmandic/automatic](https://github.com/vladmandic/automatic/), because it only uses PyTorch, and doesn't require `xformers` or `bitsandbytes`, which is good for ROCm devices. Impressive work.

## Prerequisites

### [Install AMDGPU driver with ROCm](https://docs.amd.com/bundle/ROCm-Installation-Guide-v5.5/page/How_to_Install_ROCm.html)

### [A working Automatic installation](https://are-we-gfx1100-yet.github.io/automatic/)

### A set of images for training

You can find a training image set from https://replicate.com/cloneofsimo/lora-training, but the training script doesn't support transparent images at the moment and you need to convert them to `.jpg`, using ImageMagick or something like that.

## Train

Let's start a [vladmandic/automatic](https://github.com/vladmandic/automatic/) instance first in the background, whose API will be called later.

```bash
# activate venv
cd automatic
source venv/bin/activate

# downgrade diffusers for compatibility with kohya sd scripts
# https://github.com/kohya-ss/sd-scripts/issues/402
# edit requirements.txt so that it won't be restored by launch.py
sed -i "s/^diffusers.*$/diffusers==0.14.0/g" requirements.txt
pip install diffusers==0.14.0

# train LoRA with tag and input specified via arguments
# it will do resizing, cropping, tagging and basically everything for you
# https://github.com/vladmandic/automatic/blob/master/cli/train/train.py#L80
python3 cli/train/train.py --type lora --name pokemon --tag pokemon --input datasets/pokemon_jpg/

# the trained LoRA will be located in models/Lora/
```
