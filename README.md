Edge-AI Waste Classification on OpenMV H7

ECE 4332 / ECE 6332 — AI Hardware Design and Implementation
Fall 2025

🧭 Overview

This repository contains the code, scripts, and documentation for our AI Hardware course project: Edge-AI Waste Classification on OpenMV H7.
We build an end-to-end embedded ML pipeline (dataset → training → deployment → live inference) and deploy a 4-class waste image classifier that runs fully on-device on OpenMV H7 (STM32H7 Cortex-M7 @ 480 MHz) using Edge Impulse + OpenMV IDE. 

Target classes (4): paper / plastic / metal / glass (TrashNet). 

🗂 Folder Structure

data/ – dataset organization (or pointers to TrashNet / processed assets)

docs/ – proposal and documentation

presentations/ – midterm/final slides (see final deck)

report/ – final written materials (optional; final report is also summarized in this README)

src/ – source code (OpenMV on-device inference script + any preprocessing/helpers)

(Your repo uses exactly these folders in the root, so this section matches your actual structure.)

🧑‍🤝‍🧑 Team

Team name: Circuit Board Layout 

FINAL_PRESENTATION

Members: Mengzi Cheng, Shuai Tu, Sirui You, Xueyi Zhang 

FINAL_PRESENTATION

📋 Project Artifacts

This repo contains (or will contain) the following deliverables:

Trained model artifacts (trained.tflite, labels.txt) exported for OpenMV deployment 

FINAL_PRESENTATION

OpenMV on-device inference script (camera capture → preprocessing → TFLite inference → FPS/labels output) 

FINAL_PRESENTATION

Presentation slides (final) under presentations/ 

FINAL_PRESENTATION

Report content (this README + docs/ / report/ as needed)

🧪 This Team’s Project
Motivation

Fast & reliable waste sorting needs real-time decisions near the bin. Cloud inference adds latency, requires connectivity, and introduces privacy concerns. Running the model directly on an MCU enables low-cost, low-power, standalone waste classification. 

FINAL_PRESENTATION

Goals & Success Metrics

Goal: Build an end-to-end edge AI pipeline and deploy a live camera waste classifier on OpenMV H7, fully on-device. 

FINAL_PRESENTATION

Metrics:

Accuracy (validation accuracy + confusion matrix)

Latency/FPS (real-time speed on OpenMV H7)

Memory footprint (RAM/Flash within MCU limits)

Robustness (works across lighting/background with real objects) 

FINAL_PRESENTATION

📦 Dataset & Preprocessing
Dataset

We use TrashNet (single-image 4-class waste classification: glass/metal/paper/plastic), 1,984 images, RGB 512×384. 

FINAL_PRESENTATION

Preprocessing

Input to model: resize + grayscale, 96×96 

FINAL_PRESENTATION

Online preprocessing: keep aspect ratio; convert 96×96 grayscale into normalized feature vector (reported ~4 ms, ~4 KB peak RAM for the preprocessing step). 

FINAL_PRESENTATION

🧠 Model & Training (Edge Impulse)

We use Edge Impulse’s Transfer Learning (Images) pipeline with MobileNetV2 backbone (96×96, width multiplier 0.35, dropout 0.1). 

FINAL_PRESENTATION

Training setup:

Training cycles: 45

Batch size: 32

Learning rate: 0.0005

Validation split: 20%

Data augmentation enabled 

FINAL_PRESENTATION

Validation performance:

Accuracy: 72.1%, loss: 0.66, AUC: 0.92

Weighted Precision/Recall/F1: ≈ 0.72

Paper is easiest (≈89.7% recall); glass & plastic are harder. 

FINAL_PRESENTATION

🚀 HowTo: Run the Software on the Hardware Platform (OpenMV H7)

This section is written so a teammate/TA can reproduce the on-device demo.

Requirements

Hardware: OpenMV H7 + USB cable (for power + serial) 

FINAL_PRESENTATION

Software: OpenMV IDE, Edge Impulse 

FINAL_PRESENTATION

Model files: trained.tflite, labels.txt exported from Edge Impulse 

FINAL_PRESENTATION

Step 1 — Export the model from Edge Impulse

In Edge Impulse:

Go to Deployment → OpenMV

Export OpenMV firmware package that includes .tflite + labels.txt 

FINAL_PRESENTATION

Step 2 — Copy model files to the OpenMV board

Connect OpenMV to your computer via USB

Copy trained.tflite and labels.txt to OpenMV’s file system

Make sure filenames match what the Python script loads 

FINAL_PRESENTATION

Step 3 — Run the OpenMV inference script

In OpenMV IDE:

Open the on-device script under src/ (the script should load trained.tflite + labels.txt).

Run the script. The pipeline should:

Initialize camera (example setting: RGB565, QVGA, 240×240 crop) 

FINAL_PRESENTATION

Load model using ml.Model() and run continuous inference

Print per-class probabilities + FPS to the serial monitor 

FINAL_PRESENTATION

Step 4 — Stabilize output (majority vote)

To reduce jitter, we apply a sliding-window majority vote:

Push each frame’s top-1 label into a fixed window (VOTE_WINDOW)

Output the most frequent label as the final prediction

Overlay “label + avg_score” on the live image 

FINAL_PRESENTATION

📈 Deployment & Results (On-device)
On-device performance (resource + latency)

Inference time: ~36 ms / image

Peak RAM: ~215 KB

Flash: ~536 KB

Feasible for OpenMV H7 real-time deployment 

FINAL_PRESENTATION

Live demo cases (majority-vote stabilized)

All cases below were demonstrated with live camera inference on OpenMV H7:

Paper tissue → Paper, avg_score ≈ 0.99, speed ~3.17 FPS 

FINAL_PRESENTATION

Plastic lid → Plastic, avg_score ≈ 0.99, speed ~3.17 FPS 

FINAL_PRESENTATION

Aluminum foil ball → Metal, avg_score ≈ 0.97, speed ~3.17 FPS (more challenging due to reflections) 

FINAL_PRESENTATION

Glass cup → Glass, avg_score ≈ 0.91, speed ~3.17 FPS (hardest due to transparency/lighting) 

FINAL_PRESENTATION

Summary: Paper & plastic are most stable; metal is more variable (reflections); glass is hardest (transparency & background/lighting sensitivity). 

FINAL_PRESENTATION

⚠️ Limitations

Domain shift: TrashNet images are cleaner than real on-device scenes (lighting/background), reducing robustness. 

FINAL_PRESENTATION

Hard classes: metal & glass are sensitive to reflections/transparency → higher confusion. 

FINAL_PRESENTATION

Real-time constraint: end-to-end speed is ~3.17 FPS, bounded by capture + preprocessing + inference + display/printing overhead. 

FINAL_PRESENTATION

🔮 Future Work

Data: collect in-domain images under target lighting/background conditions with real objects. 

FINAL_PRESENTATION

Model: tune augmentation for reflections/low-light (brightness/contrast, blur, clutter) to improve metal/glass. 

FINAL_PRESENTATION

System: profile latency and optimize bottlenecks (reduce printing/overlay cost, streamline preprocessing) to improve FPS. 

FINAL_PRESENTATION

🎞️ Presentation

Final slides are in presentations/ (see FINAL_PRESENTATION.pptx). 

FINAL_PRESENTATION

📜 License

This project follows the repository’s existing license file (see LICENSE in the repo root).
