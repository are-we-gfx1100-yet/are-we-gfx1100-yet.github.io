---
title: LoRA training with SD:Next on RX 7900 XTX
date: 2023-05-16
weight: -30
tags:
- torch
- torchvision
- automatic
- lora
- training
- tutorial
---

## Prerequisites

* https://are-we-gfx1100-yet.github.io/post/automatic/

To follow this tutorial, it's recommended to use our fork of SD:Next.

The official SD:Next will be able to do so once [the PR](https://github.com/vladmandic/automatic/pull/2009) is landed.

### Prepare

Prepare you images for training and place them in a directory (e.g. `automatic/datasets/cutecats`).

## Train

In terminal window A:

```bash
cd automatic

# launch sd:next in the background for use with the train script
./webui.sh
```

In terminal window B:

```bash
cd automatic

# export HIP_VISIBLE_DEVICES if you have multiple available gpus

# `--type lyco` works too
python3 cli/train.py --type lora --name cutecats --tag cutecats --input datasets/cutecats

# training for sd-xl is also supported by adding --sdxl to cli/train.py
# and add --model to specify the sd-xl checkpoint, or you need to select it in the webui
# currently no_half_vae is used to fix the original vae, which will cost more vram and have lower performance
# so it only works on rx 7900 xtx/xt for now, unless you use the fixed vae (see pr for detail)
```

Now you will see logs about images being processed according to the steps specified in the `--process` argument.

The WebUI in the background might download BLIP and DeepDanbooru models for later interrogation.

After those you will soon see logs printed from the kohya_ss script, and the output LoRAs will be stored in `automatic/models/Lora`.

## Tweak

There are many arguments to play with (run `cli/train.py --help` for the list), and even more arguments can be altered in `cli/options.py`.

I won't go through many of them which are commonly seen in many other training tutorials, but you can definitely increase the `process.target_size` in `cli/options.py` for higher resolution of input images.
