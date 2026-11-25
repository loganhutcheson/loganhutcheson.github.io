---
layout: post
title: "roblox texture editor"
description: "A year-long build across custom Stable Diffusion LoRA training, data augmentation, Modal cloud GPU hosting, Qwen + ComfyUI, and 4-step FLUX.2 precision UV editing."
---

my hardest personal build—a year of iterating across three model generations, from training custom Stable Diffusion LoRAs from scratch to shipping a full cloud-hosted web app and finally narrowing to surgical UV editing on a fixed `585 × 559` Roblox shirt template.

<div class="image-row equal-images">
  <a href="{{ '/assets/images/roblox-clothing/version-1.png' | relative_url }}" aria-label="Full image: Version 1 custom LoRA generated shirt texture"><img src="{{ '/assets/images/roblox-clothing/version-1.webp' | relative_url }}" width="585" height="559" alt="Version 1 custom LoRA generated shirt texture"></a>
  <a href="{{ '/assets/images/roblox-clothing/version-3.png' | relative_url }}" aria-label="Full image: Version 3 watch editor and live 3D Roblox avatar preview"><img src="{{ '/assets/images/roblox-clothing/version-3.webp' | relative_url }}" width="800" height="449" alt="Version 3 watch editor and live 3D Roblox avatar preview"></a>
</div>

## version 1 — dataset, LoRA training & Modal A10G

- **dataset & augmentation:** scraped a ~500-shirt seed set of `585 × 559` Roblox templates (`416` cleaned in `shirt_set.tar`, 74.7 MB via Git LFS), generated paired `rbx_shirt` captions, and expanded it to ~8,000 training examples through [data augmentation](https://en.wikipedia.org/wiki/Data_augmentation)—shading passes, rotations, alpha-to-white compositing, and multiple-of-8 tensor padding (`592 × 560`) so a small seed set generalized without breaking UV borders.
- **custom LoRA training:** hand-rolled PyTorch / Accelerate / Diffusers rank-16 (`alpha = 32`) LoRA layers directly on Stable Diffusion 1.5 UNet `attn2` cross-attention projections (`to_q`, `to_k`, `to_v`, `to_out.0`) plus a separate Lineart ControlNet LoRA (`control_v11p_sd15_lineart`, base UNet/VAE/CLIP frozen), evaluating across 300+ saved checkpoints.
- **cloud GPU serving:** packaged inference behind a FastAPI `/generate` endpoint on **Modal A10G** GPUs (with Apple Silicon MPS locally) behind a Next.js frontend on Vercel. it proved full-template generation could hold strict UV seams, but full-shirt regeneration was too noisy when users wanted to tweak one specific detail.

<div class="image-row equal-images">
  <a href="{{ '/assets/images/roblox-clothing/lora-chainmail.png' | relative_url }}" aria-label="Full image: Custom LoRA metallic chainmail Roblox shirt texture"><img src="{{ '/assets/images/roblox-clothing/lora-chainmail.webp' | relative_url }}" width="585" height="559" alt="Custom LoRA metallic chainmail Roblox shirt texture"></a>
  <a href="{{ '/assets/images/roblox-clothing/lora-crimson.png' | relative_url }}" aria-label="Full image: Custom LoRA crimson ribbed Roblox shirt texture"><img src="{{ '/assets/images/roblox-clothing/lora-crimson.webp' | relative_url }}" width="585" height="559" alt="Custom LoRA crimson ribbed Roblox shirt texture"></a>
</div>

## version 2 — Qwen + ComfyUI + production web stack

- **model graph:** upgraded to Qwen Image Edit 2509 BF16 (Qwen 2.5-VL 7B encoder + Qwen VAE) wired through a 24-node ComfyUI workflow (`TextEncodeQwenImageEditPlus`) accepting three reference images, accelerated with an 8-step Lightning LoRA and ControlNet Union (DWPose + Depth Anything).
- **product & cloud ops:** built Google OAuth sign-in, a Prisma database with atomic credit reservation and automatic refunds on upstream errors, Stripe checkout and webhooks, and homepage GPU pre-warm calls to absorb serverless cold starts. operating it in production made clear how much latency, container caching, and reliability dominate the actual user experience.

## version 3 — FLUX.2 Klein 4B, scale-to-zero L40S & watch editor

- **targeted UV editing:** narrowed from whole-shirt generation to surgical edits with 4-step **FLUX.2 Klein 4B** through Diffusers, running identically on local Apple Silicon MPS and an on-demand **Modal L40S** container (`min_containers=0`, 5s scaledown, persistent `modal.Volume` Hugging Face cache).
- **watch geometry & masking:** built a TypeScript canvas editor with live band-wrap, band-width, and watch-size controls that project wrapped geometry across the `F / L / B / R` arm UV faces, send only a `512 × 512` crop + grayscale mask to FLUX, and composite back through the mask so surrounding shirt pixels stay untouched.
- **verification, 3D preview & sprint:** paired unit-tested UV wrap math and a 10-fixture rotation/pattern benchmark (`-48°` to `142°`) with immediate 2D template and 3D Three.js OBJ avatar previews—constraining the model with deterministic UV math and tight masks beat chasing bigger models. attended and completed [Lanturn](https://lanturn.ai/)’s Founding Sprint with this build and accepted a $100 founder’s grant.

<div class="image-row equal-images">
  <a href="{{ '/assets/images/roblox-clothing/flux2-inpaint.png' | relative_url }}" aria-label="Full image: Surgical FLUX.2 wrist-watch UV inpaint on a tuxedo shirt template"><img src="{{ '/assets/images/roblox-clothing/flux2-inpaint.webp' | relative_url }}" width="585" height="559" alt="Surgical FLUX.2 wrist-watch UV inpaint on a tuxedo shirt template"></a>
  <a href="{{ '/assets/images/roblox-clothing/version-3.png' | relative_url }}" aria-label="Full image: Interactive watch editor and live 3D Roblox avatar preview"><img src="{{ '/assets/images/roblox-clothing/version-3.webp' | relative_url }}" width="800" height="449" alt="Interactive watch editor and live 3D Roblox avatar preview"></a>
</div>

source is private.
