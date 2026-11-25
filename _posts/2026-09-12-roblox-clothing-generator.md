---
layout: post
title: "bloxstyle"
description: "A year-long build across custom Stable Diffusion LoRA training, data augmentation, Modal cloud GPU hosting, Qwen + ComfyUI, and 4-step FLUX.2 precision UV editing."
---

my longest-running personal build—a year of iterating across three model generations, from training custom Stable Diffusion LoRAs from scratch to running a serverless GPU web app and narrowing down to surgical edits on a fixed `585 × 559` Roblox shirt template.

[source](https://github.com/loganhutcheson/bloxstyle)

<div class="image-row equal-images">
  <a href="{{ '/assets/images/roblox-clothing/version-1.png' | relative_url }}" aria-label="Full image: Version 1 custom LoRA generated shirt texture"><img src="{{ '/assets/images/roblox-clothing/version-1.webp' | relative_url }}" width="585" height="559" alt="Version 1 custom LoRA generated shirt texture"></a>
  <a href="{{ '/assets/images/roblox-clothing/version-3.png' | relative_url }}" aria-label="Full image: Version 3 watch editor and live 3D Roblox avatar preview"><img src="{{ '/assets/images/roblox-clothing/version-3.webp' | relative_url }}" width="800" height="449" alt="Version 3 watch editor and live 3D Roblox avatar preview"></a>
</div>

## version 1 — dataset, custom LoRA & Modal A10G

- **dataset:** scraped ~500 Roblox shirt templates (`416` cleaned in `shirt_set.tar`) and grew them to ~8,000 examples using [data augmentation](https://en.wikipedia.org/wiki/Data_augmentation)—shading passes, rotations, and padding to keep every `585 × 559` UV border intact.
- **LoRA training:** hand-rolled rank-16 SD 1.5 cross-attention LoRA layers in PyTorch/Diffusers alongside a Lineart ControlNet LoRA, testing across 300+ saved checkpoints.
- **first cloud deploy:** served it on **Modal A10G** GPUs behind Next.js on Vercel. full-template generation held the UV seams, but regenerating the whole shirt got in the way whenever someone just wanted to tweak one detail.

<div class="image-row equal-images">
  <a href="{{ '/assets/images/roblox-clothing/lora-chainmail.png' | relative_url }}" aria-label="Full image: Custom LoRA metallic chainmail Roblox shirt texture"><img src="{{ '/assets/images/roblox-clothing/lora-chainmail.webp' | relative_url }}" width="585" height="559" alt="Custom LoRA metallic chainmail Roblox shirt texture"></a>
  <a href="{{ '/assets/images/roblox-clothing/lora-crimson.png' | relative_url }}" aria-label="Full image: Custom LoRA crimson ribbed Roblox shirt texture"><img src="{{ '/assets/images/roblox-clothing/lora-crimson.webp' | relative_url }}" width="585" height="559" alt="Custom LoRA crimson ribbed Roblox shirt texture"></a>
</div>

## version 2 — Qwen + ComfyUI on Modal

- **workflow:** moved to Qwen Image Edit 2509 BF16 wired through a 24-node ComfyUI graph (3 reference images, 8-step Lightning LoRA, and ControlNet Union with DWPose + Depth Anything).
- **Modal & warm-boot UX:** packaged the weights and ComfyUI graph onto serverless [Modal](https://modal.com/) GPUs. cold starts timed out user requests, so I added background [warm-boot pings](https://github.com/loganhutcheson/bloxstyle/commit/0d9cbd3) on page load to spin the GPU up before anyone clicked generate.
- **product stack:** added Google sign-in, Prisma credit tracking with automatic refunds on upstream errors, and Stripe checkout.

<p><a href="{{ '/assets/images/roblox-clothing/qwen-comfyui.png' | relative_url }}" aria-label="Full image: 24-node Qwen Image Edit ComfyUI workflow graph"><img src="{{ '/assets/images/roblox-clothing/qwen-comfyui.webp' | relative_url }}" width="1600" height="1274" style="width:100%;height:auto;border-radius:6px;display:block;margin:24px 0" alt="24-node Qwen Image Edit ComfyUI workflow graph"></a></p>

## version 3 — FLUX.2 Klein 4B, watch editor & outcomes

- **targeted editing:** switched to 4-step **FLUX.2 Klein 4B** running locally on Apple Silicon MPS and on scale-to-zero **Modal L40S** containers with a persistent Hugging Face volume cache.
- **watch editor & 3D preview:** built a TypeScript canvas tool that wraps watch geometry across the arm UV faces, sends only a `512 × 512` masked crop to FLUX so surrounding pixels stay untouched, and renders live 3D Three.js avatar previews.
- **outcomes:** completed [Lanturn](https://lanturn.ai/)’s Founding Sprint, accepted a **$100 founder’s grant**, and hit **60 organic Google Search clicks in 28 days** on `robloxclothing.com`.

<p><a href="{{ '/assets/images/roblox-clothing/search-impact.png' | relative_url }}" aria-label="Full image: Google Search Impact badge showing 60 clicks in 28 days"><img src="{{ '/assets/images/roblox-clothing/search-impact.webp' | relative_url }}" width="926" height="1174" style="max-width:380px;width:100%;height:auto;border-radius:8px;display:block;margin:24px 0" alt="Google Search Impact: 60 clicks in 28 days for robloxclothing.com"></a></p>
