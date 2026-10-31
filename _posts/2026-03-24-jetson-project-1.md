---
layout: post
title: "jetson posture system"
description: "Camera bring-up, sensors, and pose inference on Jetson Orin Nano."
---

camera bring-up, sensors, and pose inference on Jetson Orin Nano.

[source + setup](https://github.com/loganhutcheson/jetson-posture)

<div class="image-row">
  <img src="{{ '/assets/images/jetson/irl_setup.jpg' | relative_url }}" alt="Jetson with CSI camera and status display">
  <img src="{{ '/assets/images/jetson/posture-overlay.jpg' | relative_url }}" alt="Camera detection overlay">
</div>

## what i built

- CSI camera integration through Argus and GStreamer; I²C IMU, OLED, and GPIO feedback.
- C++20 sensor runtime with bounded queues and dropped-sample tracking.
- Python pose classification, SQLite history, and systemd services.

## what i learned

bus numbering, device-tree overlays, and pinmux state mattered as much as inference. setup notes capture those hardware fixes.

paused personal prototype. the C++ camera producer is simulated; the real camera pipeline runs in Python.
