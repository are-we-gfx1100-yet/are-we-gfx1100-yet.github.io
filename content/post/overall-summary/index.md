---
title: "Overall Summary"
---

| Feature                        | Version                 | Status   | Resource                                                                                                                                                      |
| ------------------------------ | ----------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ROCm                           | 5.5.0                   | Ready    | [How to Install ROCm](https://docs.amd.com/bundle/ROCm-Installation-Guide-v5.5/page/How_to_Install_ROCm.html)                                                 |
| official ROCm Docker           | 5.5.0 + ub22.04         | Ready    | `docker pull rocm/dev-ubuntu-22.04:5.5-complete`                                                                                                              |
| official ROCm + PyTorch Docker | 5.5.0 + 2.0.0 + ub20.04 | Ready | `docker pull rocm/pytorch:rocm5.5_ubuntu20.04_py3.8_pytorch_staging`                                                                                          |
| poorman ROCm Docker            | 5.5.0 + ub22.04         | Ready    | [rocm5.5-ub22.04-base](https://github.com/evshiron/rocm_lab/pkgs/container/rocm_lab/91582912?tag=rocm5.5-ub22.04-base)                                        |
| ROCm + PyTorch                 | 5.5.0 + 2.0.1           | Building | TBD: wheels                                                                                                                                                   |
| poorman ROCm + PyTorch Docker  | 5.5.0 + 2.0.1 + ub22.04 | Pending  | TBD: rocm5.5-ub22.04-base + wheels                                                                                                                            |
| ROCm + A1111 WebUI             | 5.5.0 + 2.0.0 + latest  | Yes      | [Detail](https://evshiron.github.io/are-we-gfx1100-yet/post/a1111-webui/) |
| ROCm + Automatic               | 5.5.0 + 2.0.0 + latest  | No       |                                                                                                                                                               |
|                                |                         |          |                                                                                                                                                               |
