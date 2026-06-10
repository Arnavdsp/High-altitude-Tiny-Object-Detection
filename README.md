# Joint ESRGAN/SwinIR + RT-DETR for Tiny Object Detection in Aerial Imagery

> **Task-Driven Super-Resolution for High-Altitude Tiny Vehicle Detection using RT-DETR, NWD Loss, ESRGAN, and SwinIR on VisDrone & AI-TOD**

---

# Project Overview

This project explores the use of **Task-Driven Super-Resolution (TDSR)** for improving **tiny object detection** in challenging aerial imagery captured from high altitudes (~6 km).

The primary objective is to enhance the detectability of extremely small vehicles and aerial targets using:

* **Super-Resolution (SR)**
* **Transformer-based Detection**
* **Joint Training**
* **Normalized Wasserstein Distance (NWD) Loss**
* **Tiny Object Datasets**

---

#  Problem Statement

At high altitudes:

* Vehicles occupy only a few pixels
* Blur and atmospheric distortions degrade visibility
* Tiny objects become difficult to localize and classify
* Traditional IoU-based losses fail for ultra-small targets

This project investigates whether:

* **Super-resolution can improve detectability**
* **Task-driven SR can outperform visual-only SR**
* **NWD loss can stabilize tiny-object localization**

---

#  Datasets Used

##  VisDrone2019-DET

Used for:

* Tiny vehicle extraction
* Blur/noise degradation experiments
* SR enhancement
* RT-DETR training

Dataset includes:

* Cars
* Vans
* Buses
* Trucks
* Pedestrians
* Cyclists

Only tiny vehicle classes were filtered for this project.

---

##  AI-TOD

AI-TOD is specifically designed for:

> **Tiny object detection in aerial imagery**

Objects are often:

* 5–15 pixels wide
* Extremely sparse
* Difficult for IoU-based detectors

Used for:

* ESRGAN/SwinIR joint training
* NWD-based localization
* Tiny-object benchmarking

---

# Core Pipeline

```text
VisDrone / AI-TOD
        ↓
Vehicle Filtering
        ↓
Rotation Augmentation
        ↓
Blur + Noise Degradation
        ↓
Super-Resolution
(ESRGAN / SwinIR)
        ↓
SAHI Slicing
        ↓
RT-DETR + NWD Loss
        ↓
Tiny Object Detection
```

---

# Key Research Components

---

#  Super-Resolution Models

## ESRGAN

Enhanced Super-Resolution GAN.

Used for:

* 2× SR
* 4× SR
* Joint training

### Strengths

* Sharp outputs
* Strong visual realism
* Good texture generation

### Weaknesses

* Hallucinates textures
* Can destabilize tiny-object classification
* Generates semantic inconsistencies at high scales

---

## SwinIR

Transformer-based super-resolution model.

### Why SwinIR was explored

Unlike ESRGAN, SwinIR:

* Preserves semantic structure better
* Uses self-attention
* Reduces hallucinated textures
* Produces more stable features for RT-DETR

### Expected Advantages

* Better precision
* Better semantic consistency
* More stable 4× SR training
* Improved AP_small

---

#  RT-DETR

Real-Time Detection Transformer.

Used because:

* Anchor-free architecture
* End-to-end transformer detection
* Strong global feature modeling
* Better tiny-object adaptability compared to traditional CNN detectors

---

# 3️⃣ SAHI (Slicing Aided Hyper Inference)

Used to improve tiny-object detection.

## Why SAHI?

Tiny objects become:

* too small in full-resolution images
* hard for detectors to focus on

SAHI slices large images into smaller tiles:

```text
Large Image
     ↓
Multiple Overlapping Crops
     ↓
Detection on Crops
     ↓
Merge Predictions
```

This effectively:

* enlarges tiny objects relative to the detector input
* improves recall significantly

---

#  NWD Loss (Normalized Wasserstein Distance)

One of the most important components of this project.

---

#  Problem with IoU for Tiny Objects

For tiny objects:

```text
1–2 pixel shift
```

can reduce IoU dramatically.

Example:

```text
GT box: 6×6
Pred box shifted by 2 px
```

IoU may become nearly zero.

This creates:

* unstable gradients
* poor localization learning

---

# ✅ NWD Solution

NWD treats bounding boxes as:

```text
Gaussian distributions
```

instead of rectangles.

Bounding box:

[
(cx, cy, w, h)
]

becomes:

[
\mathcal N(\mu,\Sigma)
]

where:

[
\Sigma=
\begin{bmatrix}
w^2/12 & 0 \
0 & h^2/12
\end{bmatrix}
]

\Sigma=\begin{bmatrix}w^2/12&0\0&h^2/12\end{bmatrix}

---

# Why NWD Helps

NWD provides:

* smoother gradients
* stable localization
* robustness to tiny shifts
* better tiny-object learning

Especially important for:

* AI-TOD
* VisDrone tiny vehicles
* high-altitude aerial imagery

---

#  Loss Functions Used

---

#  Pixel Loss

[
L_{pixel}=|SR-HR|
]

L_{pixel}=|SR-HR|

### Purpose

* Preserve spatial structure
* Maintain geometric consistency

---

#  Perceptual Loss

[
L_{perc}=|\phi(SR)-\phi(HR)|
]

L_{perc}=|\phi(SR)-\phi(HR)|

Uses VGG feature extraction.

### Purpose

* Preserve semantic structure
* Maintain object recognizability

---

#  Adversarial Loss

GAN-based realism optimization.

### Purpose

* Generate sharper textures
* Improve perceptual realism

### Limitation

Can hallucinate textures for ultra-tiny objects.

---

#  Combined Loss

[
L_{total}
=========

L_{cls}
+
L_{nwd}
+
\lambda L_{sr}
]

L_{total}=L_{cls}+L_{nwd}+\lambda L_{sr}

---

# Joint Training

One of the main research goals.

Instead of training:

* SR separately
* Detector separately

the system jointly trains:

```text
ESRGAN/SwinIR
        +
RT-DETR
```

This allows:

* detector gradients to influence SR
* SR to become task-aware
* optimization for detectability instead of visual beauty

---

# Experimental Observations

---

# VisDrone 2×

* Stable convergence
* Significant recall improvement
* Improved localization
* Reduced false negatives

---

# VisDrone 4×

* Strong localization performance
* Better tiny-object reconstruction
* Improved NWD alignment

---

# AI-TOD 2×

* Most stable training
* Best semantic consistency
* Strong recall gains

---

# AI-TOD 4×

Observed:

* decreasing NWD loss
* unstable classification loss
* slight increase in total loss

### Why?

Aggressive 4× SR introduced:

* hallucinated textures
* semantic inconsistencies
* classification instability

while still improving:

* localization quality
* NWD alignment

This demonstrated an important research insight:

> Excessive SR magnification can improve localization while simultaneously degrading semantic stability for ultra-tiny objects.

---

#  Visualizations

The project includes:

* Training loss curves
* NWD convergence plots
* SR quality comparisons
* Joint training summaries
* ESRGAN output visualizations

---

#  Technologies Used

* Python
* PyTorch
* OpenCV
* Ultralytics RT-DETR
* SAHI
* ESRGAN
* SwinIR
* NumPy
* Matplotlib
* Kaggle
* Google Colab
* Lightning AI

---

# 💻 Hardware Used

Experiments were tested on:

* NVIDIA T4
* NVIDIA L4
* NVIDIA A10G

---

# 🚀 GPU Insights

## Best GPUs for this project

| GPU  | Performance                   |
| ---- | ----------------------------- |
| T4   | baseline                      |
| L4   | strong inference              |
| A10G | best overall for RT-DETR + SR |

---

#  Repository Structure

```text
project/
│
├── datasets/
├── degraded/
├── augmented/
├── sr_outputs/
├── checkpoints/
├── visualizations/
├── notebooks/
├── runs/
├── weights/
└── README.md
```

---

#  Key Research Insights

---

##  Tiny-object detection is recall-limited

Most failures occur because:

* tiny targets are missed
* not because of wrong classification

---

## NWD outperforms IoU for tiny objects

NWD provides:

* smoother gradients
* stable localization
* better tiny-object learning

---

##  Task-driven SR is more important than visual SR

The detector cares about:

* feature consistency
* semantic structure

NOT:

* photorealistic texture generation

---

## 4️⃣ SwinIR may outperform ESRGAN for AI-TOD

Because:

* transformers preserve global structure
* reduced hallucination
* more stable semantic representations

---

#  Future Work

* SwinIR + RT-DETR joint training
* Real-ESRGAN experiments
* Detection-aware perceptual loss
* Dynamic SR scaling
* Adaptive NWD weighting
* Transformer-based SR optimization
* AP_small benchmarking

---

#  Citation

If this work helps your research:

```bibtex
@project{tiny_object_sr_rtdetr,
  title={Task-Driven Super-Resolution for Tiny Object Detection using ESRGAN/SwinIR + RT-DETR + NWD},
  author={Arnav Deshpande},
  year={2026}
}
```

---

#  Author

## Arnav Deshpande

* IIT Indore
* Machine Learning
* Computer Vision
* Tiny Object Detection
* Aerial AI Systems
* RT-DETR Research
* Super-Resolution Research

---

#  Final Goal

The ultimate objective of this project is:

> Improving high-altitude tiny-object detection using task-driven super-resolution and transformer-based localization for real-world aerial surveillance systems.
