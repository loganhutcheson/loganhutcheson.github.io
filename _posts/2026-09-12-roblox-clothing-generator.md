---
layout: post
title: "Roblox clothing generator"
---

[github.com/loganhutcheson/roblox-clothing-generator](https://github.com/loganhutcheson/roblox-clothing-generator)

<div class="image-row equal-images">
  <img src="{{ '/assets/images/roblox-clothing/version-1.png' | relative_url }}" alt="Version 1 generated Roblox shirt texture">
  <img src="{{ '/assets/images/roblox-clothing/version-3.png' | relative_url }}" alt="Version 3 watch editor and Roblox avatar preview">
</div>

## Version 1 — SD 1.5 + custom LoRA

- **Training data:** a custom set of Roblox shirt templates expanded into image/caption pairs with the `rbx_shirt` trigger token.
- **Model:** Stable Diffusion 1.5 inpainting with a rank-16 LoRA trained on UNet cross-attention layers.
- **Structure control:** SD 1.5 Lineart ControlNet and an inpaint mask preserved the fixed 585 × 559 Roblox template layout.
- **Runtime:** PyTorch, Hugging Face Diffusers, Apple MPS for local work, and a Modal A10G FastAPI endpoint for hosted inference.

## Version 2 — Qwen + ComfyUI

- **Model:** Qwen Image Edit 2509 BF16 with the Qwen 2.5-VL 7B text/image encoder and Qwen VAE.
- **Workflow:** a 24-node ComfyUI graph accepted three reference images through `TextEncodeQwenImageEditPlus`.
- **Sampling:** Qwen Image Edit Lightning LoRA, eight Euler steps, CFG normalization, and the simple scheduler.
- **Control:** ControlNet Union combined DW pose preprocessing and Depth Anything guidance for layout-aware edits.

## Version 3 — FLUX.2 + watch editor

- **Model:** FLUX.2 Klein 4B through Diffusers, using four-step image editing on Apple Silicon MPS or a Modal L40S GPU.
- **Editor:** a TypeScript canvas tool converts a drawn arm guide into wrapped watch geometry on the Roblox shirt template.
- **Generation:** the browser sends a 512 × 512 crop and shape mask; FLUX refines the watch, then masked compositing protects the rest of the texture.
- **Preview:** Next.js, React, Three.js, and an OBJ avatar provide immediate 2D texture and 3D clothing previews.
