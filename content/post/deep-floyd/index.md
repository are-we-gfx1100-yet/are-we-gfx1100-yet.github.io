---
title: Run deep-floyd/IF on RX 7900 XTX
date: 2023-05-17
---

## Prerequisites

Download and install the following prebuilt wheels into your venv:

* `torch`: https://github.com/evshiron/rocm_lab/actions/runs/4981092645
* `torchvision`: https://github.com/evshiron/rocm_lab/actions/runs/4980987375

## Example

The following script is modified from https://github.com/deep-floyd/IF/blob/develop/README.md:

* load local models instead of fetching from Hugging Face
* resize stage 2 image from 256x256 to 240x240 to avoid HIP out of memory

```python
from diffusers import DiffusionPipeline
from diffusers.utils import pt_to_pil
import torch

# stage 1
stage_1 = DiffusionPipeline.from_pretrained("./IF-I-XL-v1.0", variant="fp16", torch_dtype=torch.float16)
stage_1.enable_model_cpu_offload()

# stage 2
stage_2 = DiffusionPipeline.from_pretrained(
    "./IF-II-L-v1.0", text_encoder=None, variant="fp16", torch_dtype=torch.float16
)
stage_2.enable_model_cpu_offload()

# stage 3
#safety_modules = {"feature_extractor": stage_1.feature_extractor, "safety_checker": stage_1.safety_checker, "watermarker": stage_1.watermarker}
safety_modules = {}
stage_3 = DiffusionPipeline.from_pretrained("./stable-diffusion-x4-upscaler", **safety_modules, torch_dtype=torch.float16)
stage_3.enable_model_cpu_offload()

prompt = 'a photo of a doge holding an AMD graphics card saying "wow such rocm"'

# text embeds
prompt_embeds, negative_embeds = stage_1.encode_prompt(prompt)

generator = torch.manual_seed(0)

# stage 1
image = stage_1(prompt_embeds=prompt_embeds, negative_prompt_embeds=negative_embeds, generator=generator, output_type="pt").images
pt_to_pil(image)[0].save("./if_stage_I.png")

# stage 2
image = stage_2(
    image=image, prompt_embeds=prompt_embeds, negative_prompt_embeds=negative_embeds, generator=generator, output_type="pt"
).images
pt_to_pil(image)[0].save("./if_stage_II.png")

pil_image = pt_to_pil(image)[0].resize((240, 240))

# stage 3
image = stage_3(prompt=prompt, image=pil_image, generator=generator, noise_level=100).images
image[0].save("./if_stage_III.png")
```

## Performance

* Stage 1: 100/100 [02:01<00:00,  1.21s/it]
* Stage 2: 50/50 [00:31<00:00,  1.57it/s]
* Stage 3: 75/75 [00:09<00:00,  7.71it/s]

## Conclusions

After installing our pre-built PyTorch, we were able to successfully run the three-step example script of [deep-floyd/IF](https://github.com/deep-floyd/IF) on RX 7900 XTX. 

However, we also found that there is still some gap in the memory management of ROCm devices compared to CUDA devices that can use xFormers, which requires some compromise on image size.

Hopefully, [deep-floyd/IF](https://github.com/deep-floyd/IF) will be integrated into existing Stable Diffusion solutions and employ technologies like attention optimization and tiled VAE, which should further unleash its power.
