---
title: Easy SD:Next on RX 7900 XTX
date: 2023-05-15
weight: -80
tags:
- torch
- torchvision
- automatic
- stable-diffusion
- tutorial
---

## Prerequisites

* https://are-we-gfx1100-yet.github.io/post/a1111-webui/#prerequisites

## Install the official SD:Next

```bash
git clone https://github.com/vladmandic/automatic
cd automatic
# do not export TORCH_COMMAND, as it will be automatically handled for navi 3x
./webui.sh

# as of 2023-08-14, the commands below are no longer needed

cat << __EOF > launch.sh
#!/usr/bin/env bash

# update the index if yours is not the first gpu listed in rocminfo
export HIP_VISIBLE_DEVICES=0
export HSA_OVERRIDE_GFX_VERSION=11.0.0

export TORCH_COMMAND=torch torchvision --index-url https://download.pytorch.org/whl/rocm5.6
export TENSORFLOW_PACKAGE=tensorflow==2.12.0

./webui.sh
__EOF

chmod +x launch.sh

./launch.sh
```

## Install our fork of SD:Next

```bash
git clone https://github.com/are-we-gfx1100-yet/automatic
cd automatic
# do not export TORCH_COMMAND, as it will be automatically handled for navi 3x
./webui.sh
```

See also:

* https://github.com/are-we-gfx1100-yet/automatic
* [What is this?](https://github.com/are-we-gfx1100-yet/automatic/discussions/1)

## Troubleshooting

* https://are-we-gfx1100-yet.github.io/post/a1111-webui/#troubleshooting
