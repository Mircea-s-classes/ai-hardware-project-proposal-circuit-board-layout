[![Review Assignment Due Date](https://classroom.github.com/assets/review-assignment-due-date-f15d667ab976d596c.svg)](https://classroom.github.com/a/v3c0XywZ)
# Edge-AI Waste Classification on OpenMV H7
ECE 4332 / ECE 6332 — AI Hardware Design and Implementation  
Fall 2025

## 🧭 Overview
This repository contains the code, scripts, and documentation for our AI Hardware course project: **Edge-AI Waste Classification on OpenMV H7**.

We build an end-to-end embedded ML pipeline (**dataset → training → deployment → live inference**) and deploy a **4-class waste image classifier** that runs fully on-device on **OpenMV H7 (STM32H7 Cortex-M7 @ 480 MHz)** using **Edge Impulse + OpenMV IDE**.

**Target classes (4):** paper / plastic / metal / glass (TrashNet)

---

## 🗂 Folder Structure
- `data/` – dataset organization and/or pointers to TrashNet / processed assets  
- `docs/` – project proposal and supporting documentation  
- `presentations/` – midterm/final presentation slides  
- `report/` – optional written report materials (final report is also summarized in this README)  
- `src/` – source code (OpenMV on-device inference script + helpers)

---

## 🧑‍🤝‍🧑 Team
- **Team name**: Circuit Board Layout  
- **Members**: Mengzi Cheng, Shuai Tu, Sirui You, Xueyi Zhang  

---

## 📋 Project Artifacts
This repo contains (or will contain) the following deliverables:
1. **On-device inference code** for OpenMV (camera capture → preprocessing → TFLite inference → FPS/labels output) under `src/`  
2. **Model artifacts** exported for OpenMV deployment (`trained.tflite`, `labels.txt`)  
3. **Slides** (final) under `presentations/`  
4. **Documentation** (proposal + notes) under `docs/`

---

## 🎯 Motivation & Goals
### Motivation
Real-world waste sorting benefits from **fast, low-cost, standalone** classification near the bin. Cloud inference adds latency, needs connectivity, and can introduce privacy concerns. Running the model directly on an MCU enables **low-power, always-on edge intelligence**.

### Goals & Success Metrics
- **Goal**: Build and demonstrate a full edge AI pipeline and deploy a live camera waste classifier on OpenMV H7, fully on-device.
- **Metrics**:
  - **Accuracy** (validation accuracy + confusion matrix)
  - **Latency / FPS** (real-time speed on OpenMV H7)
  - **Memory footprint** (RAM/Flash within MCU limits)
  - **Robustness** (works in real scenes with varied lighting/backgrounds)

---

## 📦 Dataset & Preprocessing
### Dataset (TrashNet)
- Task: single-image multi-class classification of household waste  
- Classes (4): glass, metal, paper, plastic  
- Image resolution: 512 × 384 (RGB)  
- Total number of images: 1,984  

### Preprocessing
- Model input: **resize + grayscale**, **96×96**
- On-device: camera frame → crop/resize → grayscale → normalized input tensor

---

## 🧠 Model & Training (Edge Impulse)
We use Edge Impulse’s **Transfer Learning (Images)** pipeline with a **MobileNetV2** backbone.

**Model settings**
- Backbone: MobileNetV2, input 96×96, width multiplier 0.35, dropout 0.1  
- Output: 4 classes (glass, metal, paper, plastic)

**Training settings**
- Training cycles: 45  
- Batch size: 32  
- Learning rate: 0.0005  
- Validation split: 20%  
- Data augmentation: enabled  

**Validation performance**
- Accuracy: **72.1%**, Loss: 0.66  
- AUC: 0.92  
- Weighted Precision/Recall/F1: ≈ 0.72  
- Paper is easiest (≈89.7% recall); glass and plastic remain harder.

---

## 🚀 How to Use This Repository
If you want to **run our code or reproduce the on-device demo**, start from:

1. **Quick Start (OpenMV H7 demo)** below  
2. The code under `src/` (OpenMV inference script + any helpers)

---

## ⚡ Quick Start (OpenMV H7 Live Demo)
### Requirements
- Hardware: **OpenMV H7** + USB cable  
- Software: **OpenMV IDE**  
- Model files: `trained.tflite`, `labels.txt` (exported from Edge Impulse)

### Step 1 — Export the Model from Edge Impulse
In Edge Impulse:
- Go to **Deployment → OpenMV**
- Export the OpenMV firmware/model package (includes **`.tflite + labels.txt`**)

### Step 2 — Copy Model Files to the OpenMV Board
- Connect OpenMV through USB
- Copy `trained.tflite` and `labels.txt` to OpenMV’s file system
- Verify filenames match what your script loads

### Step 3 — Run the Inference Script in OpenMV IDE
In OpenMV IDE:
1. Open the main script under `src/` (the script should load `trained.tflite` + `labels.txt`)
2. Run the script

Typical runtime steps in the script:
- Initialize camera (e.g., RGB565, QVGA, 240×240 crop)
- Load model using `ml.Model()`
- Run continuous inference loop
- Print class probabilities + FPS via serial monitor / console

### Step 4 — Stabilize Output (Majority Vote)
To reduce jitter, we apply a **sliding-window majority vote**:
- Push each frame’s top-1 label into a fixed-length window
- Output the most frequent label as the final prediction
- Report average confidence for the voted label

### Troubleshooting
- **“Model file not found”** → confirm `trained.tflite` / `labels.txt` are on the board and filenames match  
- **Low FPS** → reduce extra printing/overlays, profile preprocessing, use smaller ROI/crop  
- **Unstable predictions** → increase vote window length, improve lighting, reduce background clutter  

---

## 📈 Deployment & Results (On-device)
### On-device performance
- Inference time: **~36 ms / image**  
- Peak RAM usage: **~215 KB**  
- Flash usage: **~536 KB**  
- Live stream speed (end-to-end): **~3.17 FPS** (capture + preprocessing + inference + display/printing)

### Live demo cases (majority-vote stabilized)
- **Paper tissue** → Paper, avg_score ≈ 0.99, ~3.17 FPS  
- **Plastic lid** → Plastic, avg_score ≈ 0.99, ~3.17 FPS  
- **Aluminum foil ball** → Metal, avg_score ≈ 0.97, ~3.17 FPS (more challenging due to reflections)  
- **Glass cup** → Glass, avg_score ≈ 0.91, ~3.17 FPS (hardest due to transparency & lighting)

**Summary**: Paper & plastic are most stable; metal is more variable (reflections); glass is hardest (transparency/background sensitivity).

---

## ⚠️ Limitations
- **Domain shift**: TrashNet images are cleaner than real on-device scenes; lighting/background differences reduce robustness.
- **Hard classes**: metal & glass are sensitive to reflections/transparency → higher confusion vs. paper/plastic.
- **Real-time constraint**: speed is limited by capture + preprocessing + inference + display/printing overhead.

---

## 🔮 Future Work
- **Data**: collect in-domain images under multiple lighting/backgrounds with real objects.
- **Model**: tune augmentation for reflections/low-light (brightness/contrast, blur, background clutter), especially for metal/glass.
- **System**: profile latency breakdown and optimize bottlenecks (reduce printing/overlay cost, streamline preprocessing) to improve FPS and user experience.

---

## 🎞️ Presentation
- Final slides are in `presentations/` (see `FINAL_PRESENTATION.pptx`)

---

## 📜 License
See `LICENSE` in the repository root.
