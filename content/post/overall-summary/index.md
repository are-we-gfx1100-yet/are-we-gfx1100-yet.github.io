---
title: Overall Summary
date: 2023-05-04
weight: -100
---

## [How to Install ROCm](https://docs.amd.com/bundle/ROCm-Installation-Guide-v5.5/page/How_to_Install_ROCm.html)

## Prebuild wheels

https://github.com/evshiron/rocm_lab/releases

## Recommended

| Feature                          | Resource                                                                                 |
| -------------------------------- | ---------------------------------------------------------------------------------------- |
| ROCm + A1111 WebUI               | [Step by Step Tutorial](https://are-we-gfx1100-yet.github.io/post/a1111-webui/) |
| ROCm + Automatic                 | [Easy Tutorial](https://are-we-gfx1100-yet.github.io/post/automatic/)           |
| ROCm + Automatic's LoRA training | [Tutorial](https://are-we-gfx1100-yet.github.io/post/lora-training/)            |
| ROCm + deep-floyd/IF             | [Tutorial](https://are-we-gfx1100-yet.github.io/post/deep-floyd/)               |
| ROCm + tortoise-tts              | [Tutorial](https://github.com/evshiron/rocm_lab/issues/1)                                |
| ROCm + VisualGLM                 | [Tutorial](https://are-we-gfx1100-yet.github.io/post/visual-glm/)               |

## Proofs of concepts

| Feature                   | Version                            | Resource                                                                                                                           |
| ------------------------- | ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| ROCm Docker               | 5.5.0 + ub22.04                    | [rocm5.5-ub22.04-base](https://github.com/evshiron/rocm_lab/pkgs/container/rocm_lab/91582912?tag=rocm5.5-ub22.04-base)             |
| ROCm + PyTorch Docker     | 5.5.0 + 2.0.1 + ub22.04            | [rocm5.5-ub22.04-torch2.0.1](https://github.com/evshiron/rocm_lab/pkgs/container/rocm_lab/91878617?tag=rocm5.5-ub22.04-torch2.0.1) |
| ROCm + A1111 WebUI Docker | 5.5.0 + 2.0.1 + camenduru/v2.1     | [rocm5.5-a1111-webui](https://github.com/evshiron/rocm_lab/pkgs/container/rocm_lab/91995157?tag=rocm5.5-a1111-webui)               |
| ROCm + Automatic Docker   | 5.5.0 + 2.0.1 + evshiron/v20230512 | [rocm5.5-automatic](https://github.com/evshiron/rocm_lab/pkgs/container/rocm_lab/92568064?tag=rocm5.5-automatic)                   |

## Official resources

It's worth noting that the official Docker images don't support RX 7000 series out of the box.

| Feature                | Resource                                                                                                      |
| ---------------------- | ------------------------------------------------------------------------------------------------------------- |
| ROCm                   | [How to Install ROCm](https://docs.amd.com/bundle/ROCm-Installation-Guide-v5.5/page/How_to_Install_ROCm.html) |
| PyTorch Nightly        | pip3 install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/rocm5.5  |
| Tensorflow Nightly     | http://ml-ci.amd.com:21096/job/tensorflow/job/nightly-rocmfork-develop-upstream/job/nightly-build-whl/        |
| ROCm Ubuntu Docker     | https://hub.docker.com/r/rocm/dev-ubuntu-22.04                                                                |
| ROCm PyTorch Docker    | https://hub.docker.com/r/rocm/pytorch                                                                         |
| ROCm Tensorflow Docker | https://hub.docker.com/r/rocm/tensorflow                                                                      |
