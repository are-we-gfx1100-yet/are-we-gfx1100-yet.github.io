---
title: Where are we now?
date: 2023-07-08
weight: -100
---

It has been a while since the last update. The reason is that the existing articles already cover the common use cases for regular users, and with the support for PyTorch and TensorFlow being added officially, there are fewer cases that require building from scratch. The wheels compiled using GitHub Actions on ROCm LAB are also not very useful anymore.

ROCm 5.6 was released a few days ago, but the announcement did not mention support for Windows, which is disappointing. I think it would be more appropriate to call it ROCm 5.5.2. However, the announcement did mention that support for RX 7900 XTX will be provided in the fall this year. We have tested common applications extensively over the past few months, and we can expect performance improvements in future updates (hopefully not false hopes). For applications that currently cannot run or have insufficient performance, we'll have to wait for future updates to fill in the gaps.

So, where are we now?

Regarding [stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui), we tested it on [Vlad's Automatic](https://github.com/vladmandic/automatic) using the [Vlad's System Info](https://github.com/vladmandic/sd-extension-system-info) extension. Under ROCm 5.6 with a batch size of 1, it can achieve `19it/s`, which is comparable to most of the benchmarks for RTX 4080 and RTX 3090. When the batch size is increased, the maximum can reach 28it/s, but falls short of RTX 3090. Compared to ROCm 5.5, ROCm 5.6 offers a 10-20% improvement.

As for [text-generation-webui](https://github.com/oobabooga/text-generation-webui), we have ported [GPTQ for LLaMA](https://github.com/WapaMario63/GPTQ-for-LLaMa-ROCm) and [AutoGPTQ](https://github.com/are-we-gfx1100-yet/AutoGPTQ-rocm). Running 7B, 13B, and 30B models on a PCI-E 4.0 x16 slot, the inference speeds can reach `43it/s`, `25it/s`, and `15it/s` respectively. Recently, there is a project called [ExLlama](https://github.com/turboderp/exllama), which optimized the inference code for GPTQ and comes with partial ROCm support. On RX 7900 XTX, the inference speeds can reach `76it/s`, `54it/s`, and `25it/s` respectively. Although there is still a significant gap compared to RTX 4090, it is already satisfying.

As for other projects, in short, there is still a gap in terms of performance and usability compared to RTX 3090, but there is a performance improvement in ROCm 5.6 compared to ROCm 5.5.

Regarding [Triton](https://github.com/ROCmSoftwarePlatform/triton), basic support for RX 7900 XTX has been added recently, and simple examples can be run. However, running Fused Attention still results in kernel exceptions, and the performance is concerning, achieving only 13% efficiency compared to the reference implementation.

Regarding [AITemplate](https://github.com/ROCmSoftwarePlatform/AITemplate), we have recently discovered some clues. By piecing together code snippets, we can partially run the Stable Diffusion example, but it only achieves `25it/s`.

Regarding [Flash Attention](https://github.com/ROCmSoftwarePlatform/flash-attention), it currently only supports GPUs like MI200. The Attention-related code for RX 7000 series can be found in [Composable Kernel](https://github.com/ROCmSoftwarePlatform/composable_kernel), so it can be expected that RDNA 3 will also have similar performance to CUDA's xFormers and PyTorch 2.0 in the near future.
