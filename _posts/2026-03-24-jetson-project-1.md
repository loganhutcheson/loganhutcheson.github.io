---
layout: post
title: "Jetson Project #1"
---

## Overview

This is the first real working shape of the Jetson posture pipeline: CSI camera input from the IMX219, YOLO pose inference, a posture classifier layered on top of the pose output, and a small status display that can show `good`, `okay`, or `bad`.

The recent runtime work I reviewed in `jetson-runtime` lands in three steps:

- `5111005` added the main YOLO pose camera demo path
- `ba5d21d` added the Ultralytics runtime path for live Jetson inference
- `703cee6` added the posture classifier and screen output

![Jetson Orin Nano setup with IMX219 camera and status screen]({{ '/assets/images/jetson/irl_setup.jpg' | relative_url }})

The photo above is the current setup: Jetson, IMX219 camera ribbon, and the small display showing `GOOD`.

---

## Physical Setup

At the hardware level, the project is simple:

- a Jetson board runs the Python inference loop
- an IMX219 CSI camera provides the live video stream
- a small I2C display shows a one-word posture result

The camera is not treated as a generic USB webcam. The runtime opens it through Jetson's CSI stack using Argus and GStreamer, which matters because the whole interface begins with a Jetson-specific camera pipeline instead of a plain OpenCV device index.

---

## End-to-End Interface Map

The cleanest way to understand the project is as a series of interfaces:

1. camera frame interface
2. pose detection interface
3. posture feature interface
4. posture classification interface
5. human-readable output interface

That separation is what makes the system understandable. YOLO is not deciding `good` versus `bad` directly. YOLO only produces pose detections. The classifier consumes pose-derived features. The display consumes a short status string.

---

## Camera Interface

The active script is `jetson/inference/pose_camera_demo.py`. It builds a CSI pipeline string with `nvarguscamerasrc`, `nvvidconv`, `videoconvert`, and `appsink`, then hands that string to OpenCV through `cv2.VideoCapture(..., cv2.CAP_GSTREAMER)`.

Conceptually the interface is:

- input: Jetson CSI sensor id, width, height, fps
- transport: Argus / GStreamer
- output: OpenCV BGR frame

The important detail is that the rest of the runtime never talks directly to Argus. Once the frame leaves `appsink`, the rest of the pipeline sees a normal OpenCV `frame` array. That is the first major boundary in the system.

In other words:

- Jetson camera stack owns sensor access and frame delivery
- OpenCV owns the in-memory image object used by the Python loop
- everything after that operates on standard image tensors / arrays

---

## YOLO Model Loading

The script supports two inference backends:

- OpenCV DNN with an exported `.onnx` pose model
- Ultralytics with a `.pt` pose model

The backend is selected by `infer_backend()`:

- if the model path ends in `.pt`, the runtime treats it as an Ultralytics model
- otherwise it treats it as an ONNX graph for OpenCV DNN

That means the model-loading interface is intentionally narrow:

- `--model` tells the runtime which artifact to load
- `--backend auto|opencv|ultralytics` lets the caller force or infer the loader

In the current practical Jetson path, Ultralytics is the preferred runtime. When that backend is active, the script imports `YOLO` from `ultralytics` and instantiates the model with:

```python
yolo_model = YOLO(args.model)
```

That is the second major boundary:

- the camera loop provides a BGR frame
- the Ultralytics runtime owns preprocessing, forward pass, and built-in decode/NMS
- the script converts the result into its own normalized detection schema

For the OpenCV DNN path, the interface is different:

- the script loads the ONNX graph with `cv2.dnn.readNetFromONNX(args.model)`
- it performs letterboxing itself
- it creates the blob itself
- it calls `net.forward()`
- it manually decodes the dense pose tensor into boxes and keypoints

So the project supports two model runtimes, but it deliberately converts both into one shared detection format before the rest of the code runs.

---

## Per-Frame Runtime Flow

Each frame follows the same loop:

1. read one BGR frame from the CSI pipeline
2. run pose inference
3. normalize the model output into a shared detection object
4. select the top detection as `primary_detection`
5. derive posture-oriented metrics from keypoints
6. optionally classify posture from buffered pose features
7. draw overlays, write JSONL, write video, update the display

The shared detection object is the key interface for everything downstream. It contains:

- `score`
- `bbox_xyxy`
- `keypoints`

After that, the script adds `metrics`, which are derived values such as:

- chest center
- hip center
- shoulder tilt
- torso angle
- torso length normalized by bounding-box height
- lean delta after calibration

That derived metrics layer matters because raw YOLO keypoints are still fairly low-level. The project adds a more posture-friendly interface on top of them before classification.

---

## How Camera Output Is Piped Into YOLO

The camera-to-model handoff is direct and frame-based.

For the Ultralytics path:

- `cap.read()` returns one OpenCV BGR frame
- that frame is passed to `yolo_model.predict(source=frame, ...)`
- Ultralytics performs preprocessing internally
- the returned `results[0]` object is converted into plain Python dictionaries by `decode_ultralytics_result()`

For the OpenCV DNN path:

- `cap.read()` returns one BGR frame
- the script letterboxes it to a square input size
- `cv2.dnn.blobFromImage()` converts it into the network input tensor
- `net.forward()` returns the raw pose tensor
- `decode_pose_output()` maps the output back into image coordinates

This is a good architectural decision because it isolates backend-specific logic at the model boundary. Once detections are decoded, the classifier and output code do not need to care whether the pose came from Ultralytics or OpenCV DNN.

---

## Calibration and Pose Metrics

Before classification, the script establishes a baseline torso angle for the current camera placement. It collects `calibration_frames` worth of torso-angle samples from the early part of the run and averages them into `baseline_deg`.

That means the runtime is not only asking, "what is the torso angle?" It is also asking, "how far is the current torso angle from the user's neutral baseline in this camera setup?"

This is important because posture from a fixed side camera is very sensitive to physical placement:

- camera height
- desk angle
- where the subject is sitting
- whether the camera is slightly off-center

By turning raw torso angle into `lean_delta_deg`, the runtime exposes a more useful downstream signal.

---

## Classifier Training Interface

The classifier is trained by `jetson/inference/train_posture_classifier.py`.

It does not train on pixels. It trains on pose-derived feature windows extracted from JSONL captures. The command-line interface expects labeled recordings:

```bash
python3 jetson/inference/train_posture_classifier.py \
  --good /tmp/pose_good.jsonl \
  --okay /tmp/pose_okay.jsonl \
  --bad /tmp/pose_bad.jsonl \
  --model-out /tmp/posture_classifier.json \
  --report-out /tmp/posture_classifier_report.json
```

The training data interface is therefore:

- input: JSONL records written by the pose runtime
- label source: the file path grouping chosen by the user
- unit of training: aggregated frame windows, not single raw frames
- output: a saved JSON classifier model plus an optional report

This is one of the most useful design choices in the project. It means you can:

- collect data with the live pose script
- label sessions as `good`, `okay`, or `bad`
- train a lightweight classifier without retraining YOLO

YOLO stays a generic pose estimator. The posture model becomes the user-specific layer.

---

## What The Classifier Actually Learns

`posture_classifier.py` extracts a large set of engineered features from each pose detection. The code first shoulder-aligns the coordinate system:

- shoulders define the horizontal axis
- a perpendicular axis defines the vertical body frame
- all keypoints are transformed relative to chest center and shoulder width

That gives the classifier a more stable body-centric coordinate system instead of raw image pixels.

Feature groups include:

- detection confidence and keypoint confidence summaries
- shoulder width normalized by bounding-box size
- shoulder tilt and shoulder slope
- normalized x/y positions for nose, eyes, ears, shoulders, elbows, wrists, and hips
- head center
- upper arm lengths
- forearm lengths
- elbow joint angles
- wrist and elbow spacing features

At an interface level, the classifier consumes:

- one detection dictionary
- and returns one flat numeric feature dictionary

Then training builds time windows over those feature rows. A window is averaged into a single aggregate feature vector. That makes the model less dependent on one noisy frame and more reflective of short-term posture behavior.

---

## How The Classifier Is Trained

The saved posture model is a lightweight KNN-style classifier serialized as JSON. Training computes:

- the full feature-name list
- per-feature means and standard deviations
- standardized feature vectors
- stored labeled examples
- a small validation report

At inference time the model:

- standardizes the incoming feature vector
- computes distances to stored training examples
- takes the nearest `k` examples
- converts weighted votes into class probabilities

So the classifier interface is:

- input: aggregated posture feature vector
- output: probabilities over `good`, `okay`, and `bad`

This is intentionally transparent. It is much easier to inspect and tune than a second opaque neural net.

---

## Live Classification Interface

During live inference, the script loads the saved posture model with `load_posture_model(args.posture_model)`.

From there, each valid pose frame is handled like this:

1. `extract_posture_features(primary_detection)`
2. append features into `PostureWindowBuffer`
3. when enough frames are buffered, aggregate the window
4. run `predict_posture(...)`
5. smooth the probabilities with `PostureSmoother`
6. emit a stable `posture_result`

That smoothing step is important. Without it, a small posture classifier will flicker badly frame to frame. With smoothing, the display and overlays behave more like a user-facing status channel than a debugging trace.

The live result object includes:

- `label`
- `confidence`
- `probabilities`
- raw unsmoothed label/confidence/probabilities

That makes the runtime usable for both:

- human-facing feedback
- later debugging in JSONL logs

---

## Screen Output Interface

The latest display work adds a tiny text display driver in `oled_status_display.py`. Even though I casually think of it as the little LCD/OLED status screen, the code treats it as an I2C text-render target.

The interface is intentionally tiny:

- initialize an `OledStatusDisplay`
- call `write_text("good")`, `write_text("okay")`, `write_text("bad")`, or `write_text("not found")`

That means the display layer does not know anything about YOLO, keypoints, or confidence scores. It only consumes the simplest possible output contract: a short status string.

In `pose_camera_demo.py`, that contract is produced by `posture_display_text(...)`, which maps runtime state into:

- `good`
- `okay`
- `bad`
- `not found`

This is the final boundary in the project:

- inference code produces structured detections and classifier probabilities
- UI/output code consumes a tiny human-readable status

That separation is what makes the system easy to extend. A buzzer, web dashboard, serial display, or ROS topic could all subscribe to the same tiny status interface.

---

## Why This Architecture Works

The strongest part of this project is that it does not try to make one model do everything.

Instead it splits the system into clear layers:

- camera transport
- pose estimation
- posture feature extraction
- posture classification
- output / feedback

That gives each layer a specific job:

- the camera layer gets reliable frames off the IMX219
- YOLO converts pixels into a skeleton
- the feature layer converts a skeleton into posture-oriented measurements
- the classifier converts measurements into a human label
- the display converts the label into immediate feedback

That is exactly the kind of interface-first structure I want for future iterations. It should make it easier to improve any one stage without rewriting the whole project.

---

## Next Steps

The obvious next improvements are:

- collect more labeled captures for each posture class
- evaluate whether `good/okay/bad` should become more personalized per user
- compare the Ultralytics runtime against a cleaner TensorRT-friendly deployment path later
- add richer output targets beyond the small display, while keeping the same status interface

For now, though, this is the first version that feels like a real system instead of just disconnected Jetson experiments.
