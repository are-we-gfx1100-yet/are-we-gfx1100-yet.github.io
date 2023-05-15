---
title: Overall Summary
date: 2023-05-04
---

| Feature                           | Version                            | Status | Resource                                                                                                                                                             |
| --------------------------------- | ---------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ROCm                              | 5.5.0                              | Ready  | [How to Install ROCm](https://docs.amd.com/bundle/ROCm-Installation-Guide-v5.5/page/How_to_Install_ROCm.html)                                                        |
| official ROCm Docker              | 5.5.0 + ub22.04                    | Ready  | `docker pull rocm/dev-ubuntu-22.04:5.5-complete`                                                                                                                     |
| official ROCm + PyTorch Docker    | 5.5.0 + 2.0.0 + ub20.04            | Ready  | `docker pull rocm/pytorch:rocm5.5_ubuntu20.04_py3.8_pytorch_staging`                                                                                                 |
| poorman ROCm Docker               | 5.5.0 + ub22.04                    | Ready  | [rocm5.5-ub22.04-base](https://github.com/evshiron/rocm_lab/pkgs/container/rocm_lab/91582912?tag=rocm5.5-ub22.04-base)                                               |
| ROCm + PyTorch                    | 5.5.0 + 2.0.1                      | Ready  | Prebuilt wheels: [torch](https://github.com/evshiron/rocm_lab/actions/runs/4981092645) / [torchvision](https://github.com/evshiron/rocm_lab/actions/runs/4980987375) |
| poorman ROCm + PyTorch Docker     | 5.5.0 + 2.0.1 + ub22.04            | Ready  | [rocm5.5-ub22.04-torch2.0.1](https://github.com/evshiron/rocm_lab/pkgs/container/rocm_lab/91878617?tag=rocm5.5-ub22.04-torch2.0.1)                                   |
| ROCm + A1111 WebUI                | 5.5.0 + 2.0.1 + latest             | Yes    | [Step by Step Tutorial](https://evshiron.github.io/are-we-gfx1100-yet/post/a1111-webui/)                                                                             |
| ROCm + Automatic                  | 5.5.0 + 2.0.1 + latest             | Yes    | [Easy Tutorial](https://evshiron.github.io/are-we-gfx1100-yet/post/automatic/)                                                                             |
| poorman ROCm + A1111 WebUI Docker | 5.5.0 + 2.0.1 + camenduru/v2.1     | Ready  | [rocm5.5-a1111-webui](https://github.com/evshiron/rocm_lab/pkgs/container/rocm_lab/91995157?tag=rocm5.5-a1111-webui)                                                 |
| poorman ROCm + Automatic Docker   | 5.5.0 + 2.0.1 + evshiron/v20230512 | Ready  | [rocm5.5-automatic](https://github.com/evshiron/rocm_lab/pkgs/container/rocm_lab/92568064?tag=rocm5.5-automatic)                                                     |
| ROCm + Tensorflow                 | 5.5.0 + 2.10.1                     | Ready  | Prebuilt wheels: [tensorflow](https://github.com/evshiron/rocm_lab/actions/runs/4980549143)                                                                          |
