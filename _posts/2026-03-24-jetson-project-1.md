---
layout: post
title: "jetson stuff"
description: "An 8 GB Jetson Orin Nano Super repurposed across hardware bring-up, a 25 ms TensorRT driving-model server, a JetPack 7.2.2 OTA, and local LLMs."
---

one 8 GB Jetson Orin Nano Super sitting on my desk that keeps getting drafted into new jobs. it’s been a camera/sensor bench, a real-time TensorRT inference box for a comma 4, a tiny desk clock, and now a fresh JetPack 7.2.2 box for local LLM experiments.

[source + setup](https://github.com/loganhutcheson/jetson-lab)

<div class="image-row">
  <a href="{{ '/assets/images/jetson/irl_setup.jpg' | relative_url }}" aria-label="Full image: Jetson Orin Nano with CSI camera and I2C OLED display"><img src="{{ '/assets/images/jetson/irl_setup.webp' | relative_url }}" width="800" height="1067" alt="Jetson Orin Nano with CSI camera and I2C OLED display"></a>
  <a href="{{ '/assets/images/jetson/posture-overlay.jpg' | relative_url }}" aria-label="Full image: Privacy-blurred YOLO detection overlay from the CSI camera pipeline"><img src="{{ '/assets/images/jetson/posture-overlay.webp' | relative_url }}" width="800" height="450" alt="Privacy-blurred YOLO detection overlay from the CSI camera pipeline"></a>
</div>

## JetPack 7.2.2 OTA & local Qwen3.5-9B (current)

- built NVIDIA’s R36.5 → R39.2.2 image OTA and dual-slot bootloader payloads inside a privileged amd64 OrbStack Docker container on my Mac, then flashed both bootloader slots over Tailscale without pulling the NVMe.
- compiled upstream `llama.cpp` directly on the board (Ubuntu 24.04.5, kernel `6.8.12-tegra`, CUDA `13.2`, SM 8.7) and benchmarked Unsloth’s **Qwen3.5-9B** (4K context, 33/33 layers offloaded, Flash Attention, direct loading to avoid swap): **12.77 tok/s** with `IQ4_XS` (`0.47s` TTFT, `1.13 GiB` RAM headroom) vs. `11.56 tok/s` with `Q4_K_M`.
- running as a boot-enabled `systemd` user service with idle sleep, exposed privately as a browser chat over Tailscale.

## real-time TensorRT inference (`JetLink` + comma 4)

- built a TensorRT `Orin-sm87` engine for Cinque Terre Model V2 (`CTMV2`) inside a Dockerized `jetlink-server` systemd service over a direct gigabit Ethernet link (`10.55.0.1:5599`).
- benchmarked **24.7 ms mean / 24.99 ms p99** at 20 Hz (0/110 frames over the 50 ms budget) with ~4-second preload recovery on reboot. parked and backed up to a Raspberry Pi before the JetPack 7 OTA.

## Panini case OLED clock

- a 128×64 I²C OLED (`i2c-7 @ 0x3c`) running a 12-hour clock via systemd on boot, reachable anywhere over Tailscale.
- added an NTP startup-sync guard (`timedatectl NTPSynchronized=yes`) after cold boots kept starting the clock in `1969-12-31` and freezing the display.

## where it started — the posture prototype

- dual IMX219 CSI cameras through Argus/GStreamer, I²C IMU + OLED, GPIO buzzer, a C++20 bounded-queue sensor runtime, and Python YOLO pose inference.
- honestly, the posture-classifier idea itself turned out finicky and didn’t go well—so I retired it. the real payoff was the Linux hardware bring-up (device-tree overlays, bus numbering, pinmux, and systemd) that made everything after it way faster.
