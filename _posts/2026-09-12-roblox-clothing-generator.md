---
layout: post
title: "roblox texture editor"
description: "Three generations of Roblox shirt generation and precision UV texture editing: SD 1.5 LoRA, Qwen + ComfyUI, and FLUX.2 Klein 4B with a 3D avatar preview."
---

a year-long build across three model generations, narrowing from whole-shirt generation down to a focused problem: edit one clothing detail cleanly across a fixed `585 × 559` Roblox UV template without disturbing the rest of the texture.

<div class="image-row equal-images">
  <a href="{{ '/assets/images/roblox-clothing/version-1.png' | relative_url }}" aria-label="Full image: Version 1 generated Roblox shirt texture"><img src="{{ '/assets/images/roblox-clothing/version-1.webp' | relative_url }}" width="585" height="559" alt="Version 1 generated Roblox shirt texture"></a>
  <a href="{{ '/assets/images/roblox-clothing/version-3.png' | relative_url }}" aria-label="Full image: Version 3 watch editor and 3D avatar preview"><img src="{{ '/assets/images/roblox-clothing/version-3.webp' | relative_url }}" width="800" height="449" alt="Version 3 watch editor and 3D avatar preview"></a>
</div>

## version 1 — SD 1.5 + custom LoRA

- curated a custom dataset of Roblox shirt templates paired with captions using a `rbx_shirt` trigger token.
- trained a rank-16 UNet cross-attention LoRA on Stable Diffusion 1.5 inpainting, paired with Lineart ControlNet and an inpaint mask to hold the `585 × 559` UV layout.
- served generation from a FastAPI backend on Modal A10G GPUs (with Apple Silicon MPS for local runs) behind a Next.js web app on Vercel.

## version 2 — Qwen + ComfyUI + full web app

- moved to Qwen Image Edit 2509 BF16 (Qwen 2.5-VL 7B encoder + Qwen VAE) wired through a 24-node ComfyUI graph (`TextEncodeQwenImageEditPlus`) accepting three reference images.
- combined a Lightning LoRA (8 Euler steps, CFG normalization) with ControlNet Union (DW pose and Depth Anything) for layout-aware edits.
- built the surrounding product stack: Google sign-in, Prisma database with coin credits, Stripe checkout, GPU warm-boot endpoints to absorb cold starts, and an interactive 3D avatar preview.

## version 3 — FLUX.2 Klein 4B + watch editor

- switched to 4-step FLUX.2 Klein 4B editing through Diffusers on Apple Silicon MPS locally and Modal L40S GPUs remotely.
- built a TypeScript canvas tool for accessory watch mode with live band-wrap, band-width, and watch-size controls that project wrapped watch geometry across the `F / L / B / R` arm UV faces.
- sent only a targeted `512 × 512` crop and shape mask to FLUX, then composited the result back through the mask so surrounding shirt pixels stay untouched.
- rendered immediate 2D template and 3D OBJ avatar previews in Next.js, React, and Three.js.

## what i learned

training a custom LoRA and wiring a 24-node ComfyUI graph worked for full textures, but regenerating the whole template fought the user whenever they wanted one crisp detail. constraining the model to a tight crop, deterministic UV geometry, and strict masked compositing made the tool far faster and more predictable.

source is private.
