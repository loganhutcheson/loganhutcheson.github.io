---
layout: post
title: "Jetson Posture System"
---

## What I built

I turned a Jetson Orin Nano into a live posture monitor. I brought up two IMX219 CSI cameras, ran YOLO pose inference on camera frames, trained a lightweight classifier for `good`, `okay`, and `bad` posture, and sent the result to an OLED display. The final version also stores posture history and serves a local dashboard.

![Jetson Orin Nano with an IMX219 camera and status display]({{ '/assets/images/jetson/irl_setup.jpg' | relative_url }})

## How it progressed

The Git history shows the project growing in clear stages:

- `84b23bc`–`87edb39`: built the C++ sensor runtime, then added the IMU, OLED, and buzzer.
- `d21ac2a`–`3a39eb4`: brought up one camera, added YOLO pose inference, and expanded the setup to two IMX219 cameras.
- `703cee6`–`35dccc7`: added posture classification, automatic services, SQLite history, a dashboard, and final setup notes.

The repository is now organized by camera setup, inference, services, web dashboard, hardware tools, experiments, and documentation. I have retired the project for now, with the working system preserved in its final state.

## What I learned

The hardest work was at the boundaries: Jetson device trees and camera pipelines, model output and posture features, and hardware status output. Separating those pieces made debugging much easier. I also learned that a generic pose model can become a useful personal system with calibration, a small classifier, and reliable data collection—without retraining the vision model itself.
