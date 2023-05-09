---
title: Run A1111 WebUI on Ubuntu 22.04 with RX 7900 XTX
---

Yesterday I was regularly checking ROCm PRs, and surprised to discover that the ROCm 5.5.0 release notes had been merged, providing official support for my RTX 7900 XTX after a weeks-long wait. I can't wait to test it out.

## Prerequisites

### Install AMDGPU driver

```bash
# grab the latest amdgpu-install package
curl -O https://repo.radeon.com/amdgpu-install/5.5/ubuntu/jammy/amdgpu-install_5.5.50500-1_all.deb

sudo dpkg -i amdgpu-install_5.5.50500-1_all.deb

# install AMDGPU with ROCm support (which is now 5.5.0)
sudo amdgpu-install --usecase=graphics,rocm

# you should see rocm-5.5.0 here
ls -l /opt

# grant device access to current user
# log in again or reboot should guarantee it works
sudo usermod -aG video $USER
sudo usermod -aG render $USER

# check rocm info, eg. the architecture
sudo rocminfo
# nvidia-smi alike
rocm-smi
```

### Install Ubuntu dependencies

```bash
# correct me if anything is missing
sudo apt install git build-essential

sudo apt install python3-pip python3-venv python3-dev

# fix <cmath> not found
sudo apt install libstdc++-12-dev

# optional, suppress warnings from torchvision
sudo apt install libpng-dev libjpeg-dev
```

## Building

### Enter venv environment

```bash
# can be anywhere you prefer
mkdir ~/stable-diffusion
cd ~/stable-diffusion

python3 -m venv venv

# activate venv
source venv/bin/activate
```

### Compile PyTorch 2.0.1

```bash
# must be in venv
cd ~/stable-diffusion
source venv/bin/activate

curl -L -O https://github.com/pytorch/pytorch/releases/download/v2.0.1/pytorch-v2.0.1.tar.gz
tar -xzvf pytorch-v2.0.1.tar.gz

cd pytorch-v2.0.1
echo 2.0.1 > version.txt

# update the path to yours
export CMAKE_PREFIX_PATH="%HOME/stable-diffusion/venv"

export USE_CUDA=0
export PYTORCH_ROCM_ARCH=gfx1100

pip install cmake ninja
pip install -r requirements.txt
pip install mkl mkl-include
python3 tools/amd_build/build_amd.py
python3 setup.py install
```

### Test PyTorch functionality

Run `python3` and try these out:

```python
# taken from https://github.com/RadeonOpenCompute/ROCm/issues/1930
import torch
torch.cuda.is_available()
torch.cuda.device_count()
torch.cuda.current_device()
torch.cuda.get_device_name(torch.cuda.current_device())

tensor = torch.randn(2, 2)
res = tensor.to(0)
# if it crashes, check this:
# https://github.com/RadeonOpenCompute/ROCm/issues/1930
print(res)
```

You can also try this:

```bash
PYTORCH_TEST_WITH_ROCM=1 python3 test/run_test.py --verbose
```

I failed in `test/profiler/test_profiler.py`, but it doesn't affect `stable-diffusion-webui` (hopefully).

### Compile torchvision 0.15.2

```bash
# must be in venv
cd ~/stable-diffusion
source venv/bin/activate

curl -L -O https://github.com/pytorch/vision/archive/refs/tags/v0.15.2.tar.gz
tar -xzvf v0.15.2.tar.gz

cd vision-0.15.2
echo 0.15.2 > version.txt

# update the path to yours
export CMAKE_PREFIX_PATH="%HOME/stable-diffusion/venv"

export FORCE_CUDA=1

python3 setup.py install
```

### Set up stable-diffusion-webui

```bash
# must be in venv
cd ~/stable-diffusion
source venv/bin/activate

git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui

cd stable-diffusion-webui

# remove torch from requirements.txt
# idk if it is ok to skip
sed '/torch/d' requirements.txt

pip install -r requirements.txt

# not needed if only one gpu is present
export HIP_VISIBLE_DEVICES=0

python3 launch.py
```

### Conclusions

At first, I was building `pytorch` in Docker, with [rocm/composable_kernel:ck_ub20.04_rocm5.5](https://hub.docker.com/layers/rocm/composable_kernel/ck_ub20.04_rocm5.5/images/sha256-7ecc3b5e2e0104a58188ab5f26085c31815d2ed03955d66b805fc10d9e1f6873?context=explore) as the base, but I encountered Segmentation Fault and didn't have the time to try again. Hopefully [rocm/pytorch](https://hub.docker.com/r/rocm/pytorch) will update in recent days too.

Compared to ROCm 5.5 RC4 in Docker, ROCm 5.5 + bare installation does improve in stability and VRAM management. Performance can be optimized further, but I am satisfied now.

UPDATE: Using `PYTORCH_ROCM_ARCH=gfx1100` to build `torch` with support for 7000 series, and `FORCE_CUDA=1` to build `torchvision` with complete support for ROCm. You should not encounter Segmentation Fault after that.

## References

* https://gist.github.com/In-line/c1225f05d5164a4be9b39de68e99ee2b
* https://docs.amd.com/bundle/ROCm-Deep-Learning-Guide-v5.5/page/Frameworks_Installation.html
* https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/9591
* https://github.com/RadeonOpenCompute/ROCm/issues/1930
