---
title: Run A1111 WebUI on Ubuntu 22.04 with RX 7900 XTX
date: 2023-05-02
tags:
- torch
- torchvision
- a1111-webui
- stable-diffusion
- tutorial
---

Nowadays, it's much simpler to set up Stable Diffusion WebUI on RX 7900 XTX. To provide information as much as it can, the old approach is kept but it's not recommended unless you want to compile `torch` by yourself.

## The easy approach

### Prerequisites

```bash
# install the amdgpu driver with rocm support
curl -O https://repo.radeon.com/amdgpu-install/5.6/ubuntu/jammy/amdgpu-install_5.6.50600-1_all.deb
sudo dpkg -i amdgpu-install_5.6.50600-1_all.deb

# opencl might cause issues later, so skip it unless you need it
sudo amdgpu-install --usecase=graphics,rocm

# grant current user the access to gpu devices
sudo usermod -aG video $USER
sudo usermod -aG render $USER

# reboot is needed to make both driver and user group take effect
sudo reboot
```

### Install

```bash
cd to/anywhere/you/want

git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
cd stable-diffusion-webui

# set up venv
python3 -m venv venv
source venv/bin/activate

# install dependencies
pip3 install -r requirements.txt

# replace torch with the rocm one
pip3 uninstall torch torchvision
pip3 install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/rocm5.5

# download a launch script from gist, you can create it yourself if you are concerned
curl https://gist.githubusercontent.com/evshiron/8cf4de34aa01e217ce178b8ed54a2c43/raw/e5743505afe6b2a329908bbefda93d98b98940ac/launch.sh > launch.sh
```

### Launch

```bash
# launch the webui
bash launch.sh
```

You should get a log like this:

```
(venv) user@hostname:~/test/stable-diffusion-webui$ bash launch.sh 
Python 3.10.6 (main, May 29 2023, 11:10:38) [GCC 11.3.0]
Version: v1.4.1
Commit hash: f865d3e11647dfd6c7b2cdf90dde24680e58acd8
Installing clip
Installing open_clip
Cloning Stable Diffusion into /home/user/test/stable-diffusion-webui/repositories/stable-diffusion-stability-ai...
Cloning K-diffusion into /home/user/test/stable-diffusion-webui/repositories/k-diffusion...
Cloning CodeFormer into /home/user/test/stable-diffusion-webui/repositories/CodeFormer...
Cloning BLIP into /home/user/test/stable-diffusion-webui/repositories/BLIP...
Installing requirements for CodeFormer
Installing requirements
Launching Web UI with arguments: --listen --enable-insecure-extension-access --opt-sdp-attention
No module 'xformers'. Proceeding without it.
Downloading: "https://huggingface.co/runwayml/stable-diffusion-v1-5/resolve/main/v1-5-pruned-emaonly.safetensors" to /home/user/test/stable-diffusion-webui/models/Stable-diffusion/v1-5-pruned-emaonly.safetensors

100%|█████████████████████████████████████████████████████████████████████████████████████████████████| 3.97G/3.97G [06:22<00:00, 11.2MB/s]
Calculating sha256 for /home/user/test/stable-diffusion-webui/models/Stable-diffusion/v1-5-pruned-emaonly.safetensors: preload_extensions_git_metadata for 7 extensions took 0.00s
Running on local URL:  http://0.0.0.0:7860

To create a public link, set `share=True` in `launch()`.
Startup time: 385.1s (import torch: 0.7s, import gradio: 0.5s, import ldm: 0.2s, other imports: 0.3s, list SD models: 382.8s, load scripts: 0.2s, create ui: 0.2s).
6ce0161689b3853acaa03779ec93eafe75a02f4ced659bee03f50797806fa2fa
Loading weights [6ce0161689] from /home/user/test/stable-diffusion-webui/models/Stable-diffusion/v1-5-pruned-emaonly.safetensors
Creating model from config: /home/user/test/stable-diffusion-webui/configs/v1-inference.yaml
LatentDiffusion: Running in eps-prediction mode
DiffusionWrapper has 859.52 M params.
Applying attention optimization: sdp... done.
Textual inversion embeddings loaded(0): 
Model loaded in 3.6s (calculate hash: 2.4s, load weights from disk: 0.1s, create model: 0.2s, apply weights to model: 0.2s, apply half(): 0.2s, move model to device: 0.2s, calculate empty prompt: 0.1s).
```

---

## The old approach

Yesterday I was regularly checking ROCm PRs, and surprised to discover that the ROCm 5.5.0 release notes had been merged, providing official support for my RX 7900 XTX after a weeks-long wait. I can't wait to test it out.

See also [Easy vladmandic/automatic on RX 7900 XTX](https://are-we-gfx1100-yet.github.io/post/automatic/), which is recommended for a smooth experience and has more troubleshooting tips.

### Prerequisites

#### Install AMDGPU driver

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

#### Install Ubuntu dependencies

```bash
# correct me if anything is missing
sudo apt install git build-essential

sudo apt install python3-pip python3-venv python3-dev

# fix <cmath> not found
sudo apt install libstdc++-12-dev

# optional, suppress warnings from torchvision
sudo apt install libpng-dev libjpeg-dev
```

### Build

#### Enter venv environment

```bash
# can be anywhere you prefer
mkdir ~/stable-diffusion
cd ~/stable-diffusion

python3 -m venv venv

# activate venv
source venv/bin/activate
```

#### Build `torch` by yourself

##### Compile PyTorch 2.0.1

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

# set up essential environment variables
export USE_CUDA=0
export PYTORCH_ROCM_ARCH=gfx1100

# install dependencies and build & install into venv
pip install cmake ninja
pip install -r requirements.txt
pip install mkl mkl-include
python3 tools/amd_build/build_amd.py
python3 setup.py install
```

##### Test PyTorch functionality

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

##### Compile torchvision 0.15.2

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

# set up essential environment variables
export FORCE_CUDA=1

# build & install into venv
python3 setup.py install
```

### Install

```bash
# must be in venv
cd ~/stable-diffusion
source venv/bin/activate

git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui

cd stable-diffusion-webui

# remove torch from requirements.txt
# idk if it is ok to skip
sed -i '/^torch\s*$/d' requirements.txt

pip install -r requirements.txt

# not needed if only one gpu is present in rocm-smi
export HIP_VISIBLE_DEVICES=0

python3 launch.py
```

### Launch

```bash
cd ~/stable-diffusion
source venv/bin/activate

python3 launch.py
```

### Conclusion

Using `PYTORCH_ROCM_ARCH=gfx1100` to build `torch` with support for Navi 31, and `FORCE_CUDA=1` to build `torchvision` with complete support for ROCm.

### Caveats

#### `RuntimeError: Please use PyTorch built with LAPACK support`

LAPACK support should be provided by MKL, however, the process described above doesn't really include MKL.

The MKL installed by `pip install mkl mkl-include` can't be found when building `torch`. If you want MKL to be included, you should be using `conda` for these dependencies, which is another story.

If you want MKL and MAGMA to be included, here are some resources:

* https://github.com/pytorch/pytorch/blob/main/.ci/docker/ubuntu-rocm/Dockerfile
* https://github.com/evshiron/rocm_lab/pkgs/container/rocm_lab/95731066?tag=rocm5.5-focal-torch-dev

#### `PyTorch was not compiled with MAGMA support`

The UniPC sampler uses MAGMA, which is not essential and I skip it.

Just use other samplers instead.

### References

* https://gist.github.com/In-line/c1225f05d5164a4be9b39de68e99ee2b
* https://docs.amd.com/bundle/ROCm-Deep-Learning-Guide-v5.5/page/Frameworks_Installation.html
* https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/9591
* https://github.com/RadeonOpenCompute/ROCm/issues/1930

## Troubleshooting

### Image generation freezes or performs badly

* Run `rocminfo`
  * If the command is not found, reinstall the AMDGPU driver
  * If you have multiple `gfx` GPUs, export `HIP_VISIBLE_DEVICES=0` (included in my `launch.sh`) to use the first GPU only, and you can update it according to your case
  * If you have Navi 3x GPUs other than Navi 31, export `HSA_OVERRIDE_GFX_VERSION=11.0.0` to make it treated as Navi 31 and see if it works
* Run `rocm-smi`
  * Check power consumption is correct, reinstall the AMDGPU driver if it isn't
* Disable `--no-half-vae` if it's previously enabled

### `--no-half-vae` and `ring sdma0 timeout`

`--no-half-vae` can be used to avoid some black image issues. In early versions of ROCm, it will cause a lot of `ring sdma0 timeout`, where the GPU will be reset and the progress will be lost.

Nowadays, it works just fine on my end. If you encounter weird issues like resetting or freezing, disabling it is still worth a try.

### `HIP out of memory`

Currently, the SDP attention is not optimized for ROCm, where the math implementation is used, which is a bit more VRAM hungry then the other optimizations. You might switch to something like Doggettx's and stay tuned.

For very large image generation, [Tiled VAE](https://github.com/pkuliyi2015/multidiffusion-upscaler-for-automatic1111) is commonly used, which saves a lot of VRAM and you should try that. It's worth noting that a Decoder Tile Size of 128 to 160 works better in my case.

### `fatal error: 'limits' file not found`

Just run `sudo apt install libstdc++-12-dev` for the missing header files.
