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

# patch installer.py
# remove HSA_OVERRIDE_GFX_VERSION which will fail our gfx1100
sed -i "/os.environ.setdefault('HSA_OVERRIDE_GFX_VERSION', '10.3.0')/d" installer.py
```

## Launch

```bash
# launch webui with --medvram
./webui.sh --medvram
```

## Caveats

### `RuntimeError: HIP error: invalid argument`

Don't forget to run this:

```bash
# patch installer.py
# remove HSA_OVERRIDE_GFX_VERSION which will fail our gfx1100
sed -i "/os.environ.setdefault('HSA_OVERRIDE_GFX_VERSION', '10.3.0')/d" installer.py
```

### `PyTorch was not compiled with MAGMA support`

The UniPC sampler uses MAGMA, which is not essential and I skip it.

Just use other samplers instead.

### `HIP out of memory`

Go to Settings in Automatic WebUI, set "Cross-attention optimization method" to "Doggettx's" and restart the process (not "Restart Server").

Currently, SDP attention unfortunately doesn't help with RX 7900 XTX's iteration speed and is memory hungry. It's better switch back to old optimization and stay tuned.

### `ring sdma0 timeout`

The AMDGPU driver will reset the graphics card if pushed too hard.

For Tiled VAE, a Decoder Tile Size smaller than `128` will make the GPU work more stable and reach higher potential.

`--no-half-vae` might help with black image issues, but will increase the chance to reset, so use it on your own risk.

## Showcases

![images/showcase/waifu.jpg](images/showcase/waifu.jpg)
 