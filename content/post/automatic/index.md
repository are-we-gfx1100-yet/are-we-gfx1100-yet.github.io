---
title: Easy vladmandic/automatic on RX 7900 XTX
date: 2023-05-15
image: images/showcase/waifu.jpg
---

## Prerequisites

Download the following prebuilt wheels and unzip into `~/Downloads`.

* `torch`: https://github.com/evshiron/rocm_lab/actions/runs/4981092645
* `torchvision`: https://github.com/evshiron/rocm_lab/actions/runs/4980987375

## Install

```bash
git clone https://github.com/vladmandic/automatic
cd automatic

# create default venv dir
python3 -m venv venv
source venv/bin/activate

# install custom torch and torchvision
pip install ~/Downloads/torch-2.0.1+git8f7b63d-cp310-cp310-linux_x86_64.whl
pip install ~/Downloads/torchvision-0.15.2+8f7b63d-cp310-cp310-linux_x86_64.whl
```

## Launch

```bash
# launch webui with --medvram
./webui.sh --medvram
```

## Caveats

### `ring sdma0 timeout`

The AMDGPU driver will reset the graphics card if pushed too hard.

For Tiled VAE, a Decoder Tile Size smaller than `128` will make the GPU work more stable and reach higher potential.

`--no-half-vae` might help with black image issues, but will increase the chance to reset, so use it on your own risk.

## Showcases

![images/showcase/waifu.jpg](images/showcase/waifu.jpg)
 