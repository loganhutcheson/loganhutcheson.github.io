---
layout: post
title: "Jetson Runtime / Posture Pipeline"
---

<div class="image-row">
  <img src="{{ '/assets/images/jetson/irl_setup.jpg' | relative_url }}" alt="Jetson Orin Nano with an IMX219 camera and status display">
  <img src="{{ '/assets/images/jetson/posture-overlay.jpg' | relative_url }}" alt="Live camera detection with person confidence and position overlay">
</div>

## Technical stack

- **Compute:** NVIDIA Jetson Orin Nano
- **Cameras:** Two Sony IMX219 CSI modules
- **Video pipeline:** NVIDIA Argus, GStreamer, and OpenCV
- **Pose estimation:** YOLO11 Pose with Ultralytics and OpenCV DNN backends
- **Posture model:** Shoulder-aligned keypoint features, window aggregation, K-nearest neighbors, and probability smoothing
- **Sensor and feedback hardware:** MPU-6050 IMU, I2C OLED, and GPIO buzzer
- **Persistence:** SQLite posture history
- **Runtime:** Python inference and dashboard services with a C++20 sensor runtime
