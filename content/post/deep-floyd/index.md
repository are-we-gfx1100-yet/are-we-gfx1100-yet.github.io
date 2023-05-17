---
title: Run deep-floyd/IF on RX 7900 XTX
date: 2023-05-17
---

The following script is modified from https://github.com/deep-floyd/IF/blob/develop/README.md:

* load local models instead of fetching from Hugging Face
* resize stage 2 image from 256x256 to 240x240 to avoid HIP out of memory

```bash
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
