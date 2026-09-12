---
layout: post
title: "Jetson Runtime / Posture Pipeline"
---

<div class="project-meta">
<strong>STATUS</strong> &nbsp;retired / preserved<br>
<strong>STACK</strong> &nbsp;Jetson Orin Nano · dual IMX219 · YOLO Pose · Python · C++ · SQLite
</div>

## System

- **Capture:** brought up two IMX219 CSI cameras through NVIDIA Argus and GStreamer.
- **Inference:** ran YOLO Pose on live frames and normalized backend output into a shared detection schema.
- **Classification:** derived body-relative features, aggregated short time windows, and classified posture as `good`, `okay`, or `bad`.
- **Output:** wrote the current state to an I2C OLED, recorded history in SQLite, and exposed a local dashboard.

![Jetson Orin Nano with an IMX219 camera and status display]({{ '/assets/images/jetson/irl_setup.jpg' | relative_url }})

## Development timeline

- **Runtime foundation:** [bounded sensor buffers](https://github.com/loganhutcheson/jetson-runtime/commit/c3a12e9), [IMU sampling](https://github.com/loganhutcheson/jetson-runtime/commit/fbd02dd), and [OLED/buzzer output](https://github.com/loganhutcheson/jetson-runtime/commit/87edb39).
- **Vision path:** [CSI camera inference](https://github.com/loganhutcheson/jetson-runtime/commit/5111005), [Ultralytics runtime support](https://github.com/loganhutcheson/jetson-runtime/commit/ba5d21d), and [dual-camera bring-up](https://github.com/loganhutcheson/jetson-runtime/commit/3a39eb4).
- **Posture system:** [live classification](https://github.com/loganhutcheson/jetson-runtime/commit/703cee6), [SQLite history](https://github.com/loganhutcheson/jetson-runtime/commit/bd6cb86), and [dashboard analytics](https://github.com/loganhutcheson/jetson-runtime/commit/553993e).
- **Final state:** [setup documentation](https://github.com/loganhutcheson/jetson-runtime/commit/35dccc7) and [repository retirement cleanup](https://github.com/loganhutcheson/jetson-runtime/commit/35ab2e1).

## Code map

- [`pose_camera_demo.py`](https://github.com/loganhutcheson/jetson-runtime/blob/main/jetson/inference/pose_camera_demo.py) — camera pipeline, inference loop, metrics, and output routing.
- [`posture_classifier.py`](https://github.com/loganhutcheson/jetson-runtime/blob/main/jetson/inference/posture_classifier.py) — feature extraction, window aggregation, KNN inference, and smoothing.
- [`posture_history.py`](https://github.com/loganhutcheson/jetson-runtime/blob/main/jetson/inference/posture_history.py) — persistent posture events and summaries.
- [`posture_dashboard.py`](https://github.com/loganhutcheson/jetson-runtime/blob/main/jetson/web/posture_dashboard.py) — local history and analytics interface.
- [`force_imx219_cam1_dtb.sh`](https://github.com/loganhutcheson/jetson-runtime/blob/main/jetson/camera/force_imx219_cam1_dtb.sh) — repeatable single/dual IMX219 device-tree setup.
- [`POSTURE_SYSTEM_SETUP.md`](https://github.com/loganhutcheson/jetson-runtime/blob/main/docs/POSTURE_SYSTEM_SETUP.md) — final deployment and service notes.

## Where it got difficult

- **The configured camera tree was not the live camera tree.** Selecting an overlay in `extlinux.conf` and updating the DTB under `/boot` looked correct, but the Jetson still booted from the `A_kernel-dtb` / `B_kernel-dtb` partitions. Getting both cameras online required merging the official dual-IMX219 overlay into the board DTB, writing both runtime partitions, rebooting, and verifying the live tree and both Argus sensor IDs.
- **GPIO state disagreed with the overlay.** Header pin 7 appeared as `PAC.06`, but the live PADCTL register still read `0x5a`: input and tristated. The working buzzer path required a boot-time register correction to `0x0a`, plus accounting for the module's active-low logic. This was a board-state problem below the application layer.
- **Pose detections were easier than stable posture labels.** Raw keypoints moved with framing and single-frame noise. The classifier needed shoulder-aligned coordinates, normalization by body scale, short-window aggregation, confidence-weighted neighbors, and probability smoothing before `good` / `okay` / `bad` was stable enough for the display.
- **A working local server was not automatically reachable.** `jetson.local` could resolve to IPv6 while the dashboard listened only on IPv4. The final service binds to `::` with dual-stack support, and the Mac helper falls back to an SSH tunnel when direct access fails.

## Engineering takeaways

- Hardware interfaces dominated integration time: device-tree overlays, CSI sensor routing, I2C addressing, GPIO pinmux state, and service startup order all had to agree.
- Converting OpenCV DNN and Ultralytics results into one detection format kept calibration, classification, logging, and display code independent of the model backend.
- Body-relative features and per-installation calibration were more useful than raw image coordinates for a fixed-camera posture classifier.
- A small, inspectable classifier on top of a general pose model was sufficient; retraining the vision model was unnecessary.
