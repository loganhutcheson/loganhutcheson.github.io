---
layout: post
title: "bloxstyle"
description: "A year-long build across custom Stable Diffusion LoRA training, data augmentation, Modal cloud GPU hosting, Qwen + ComfyUI, and 4-step FLUX.2 precision UV editing."
---

my hardest personal build—a year of iterating across three model generations, from training custom Stable Diffusion LoRAs from scratch to shipping a full cloud-hosted web app and finally narrowing to surgical UV editing on a fixed `585 × 559` Roblox shirt template.

[source](https://github.com/loganhutcheson/bloxstyle)

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

## version 2 — Qwen + ComfyUI on Modal + production web stack

- **model graph:** upgraded to Qwen Image Edit 2509 BF16 (Qwen 2.5-VL 7B encoder + Qwen VAE) wired through a 24-node ComfyUI workflow (`TextEncodeQwenImageEditPlus`) accepting three reference images, accelerated with an 8-step Lightning LoRA and ControlNet Union (DWPose + Depth Anything).
- **Modal hosting & warm-boot UX:** learned how to package multi-gigabyte diffusion weights and a full ComfyUI graph onto serverless [Modal](https://modal.com/) GPUs—and how to keep the web app feeling responsive by firing background [warm-boot pings](https://github.com/loganhutcheson/bloxstyle/commit/0d9cbd3) on homepage mount (`warm: true`) to absorb GPU cold starts before a user clicks generate.
- **product & billing:** built Google OAuth sign-in, a Prisma database with atomic credit reservation and automatic refunds on upstream errors, and Stripe checkout and webhooks around the inference proxy.

<a href="{{ '/assets/images/roblox-clothing/qwen-comfyui.png' | relative_url }}" aria-label="Full image: 24-node Qwen Image Edit ComfyUI workflow graph"><img src="{{ '/assets/images/roblox-clothing/qwen-comfyui.webp' | relative_url }}" width="1600" height="1280" style="width:100%;height:auto;border-radius:6px;display:block;margin:24px 0" alt="24-node Qwen Image Edit ComfyUI workflow graph"></a>

## version 3 — FLUX.2 Klein 4B, watch editor & outcomes

- **targeted UV editing:** narrowed from full-shirt generation to 4-step **FLUX.2 Klein 4B** editing via Diffusers, running identically on local Apple Silicon MPS and an on-demand **Modal L40S** container (`min_containers=0`, 5s scaledown, persistent `modal.Volume` cache).
- **watch editor & 3D preview:** built a TypeScript canvas tool projecting wrapped watch geometry across the `F / L / B / R` arm UV faces, sending a targeted `512 × 512` crop + mask with strict masked compositing, unit-tested UV wrap math, a 10-fixture angle benchmark, and live Three.js OBJ avatar previews.
- **outcomes:** attended and completed [Lanturn](https://lanturn.ai/)’s Founding Sprint, accepted a **$100 founder’s grant**, and reached **60 organic Google Search clicks in 28 days** on `robloxclothing.com`.

<a href="{{ '/assets/images/roblox-clothing/search-impact.png' | relative_url }}" aria-label="Full image: Google Search Impact badge showing 60 clicks in 28 days"><img src="{{ '/assets/images/roblox-clothing/search-impact.webp' | relative_url }}" style="max-width:380px;width:100%;height:auto;border-radius:8px;display:block;margin:24px 0" alt="Google Search Impact: 60 clicks in 28 days for robloxclothing.com"></a>
