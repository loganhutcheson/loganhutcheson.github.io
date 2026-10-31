---
layout: post
title: "roblox texture editor"
description: "Guided image editing, texture compositing, and a 3D preview."
---

a clothing editor built around a narrow problem: change one detail without disturbing the rest of the texture.

<div class="image-row equal-images">
  <img src="{{ '/assets/images/roblox-clothing/version-1.png' | relative_url }}" alt="Early generated shirt texture">
  <img src="{{ '/assets/images/roblox-clothing/version-3.png' | relative_url }}" alt="Watch editor and 3D avatar preview">
</div>

## what i built

- TypeScript drawing tools that map a guide onto clothing geometry.
- masked image refinement and compositing to preserve surrounding pixels.
- React and Three.js previews for checking the result on an avatar.

## what i learned

earlier LoRA and Qwen experiments led to a smaller FLUX editing workflow. precise masks and immediate feedback made the tool more useful.

experimental; source is private.
