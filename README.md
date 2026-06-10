# High-Altitude Tiny Object Detection using RT-DETR-L, ESRGAN, SwinIR, SAHI, and NWD Loss

## Task-Driven Super-Resolution for Tiny Aerial Object Detection on VisDrone2019-DET and AI-TOD

---

## Overview

This project focuses on high-altitude tiny object detection in aerial imagery using Task-Driven Super-Resolution (TDSR), transformer-based object detection, and advanced localization losses.

The system is designed to improve the detectability of extremely small aerial targets such as vehicles and aircraft captured from high altitudes (~6 km), where objects occupy only a few pixels and are heavily affected by blur, atmospheric distortion, and low spatial resolution.

The project integrates:

* RT-DETR-L for transformer-based detection
* ESRGAN and SwinIR for super-resolution
* SAHI for slicing-assisted inference
* NWD (Normalized Wasserstein Distance) loss for tiny-object localization
* Joint training between SR and detection modules
* Comparative analysis against IoU-based localization

---

# Problem Statement

Tiny object detection in aerial imagery presents several challenges:

* Vehicles and aircraft occupy very few pixels
* Atmospheric blur and sensor noise degrade visibility
* Small localization errors severely affect IoU
* Conventional detectors struggle with tiny targets
* Standard IoU losses become unstable for ultra-small objects

This project investigates whether:

* Super-resolution can improve tiny-object detectability
* Task-driven SR can outperform visually optimized SR
* NWD loss can stabilize localization for ultra-small targets
* Transformer-based SR models outperform GAN-based SR for aerial imagery

---

# Datasets

## VisDrone2019-DET

Used for:

* Tiny vehicle extraction
* High-altitude vehicle detection
* Blur and noise degradation experiments
* Super-resolution-assisted detection
* RT-DETR-L training and evaluation

Filtered classes include:

* Cars
* Vans
* Trucks
* Buses

---

## AI-TOD

AI-TOD is specifically designed for tiny object detection in aerial imagery.

Characteristics:

* Objects often 5–15 pixels wide
* Sparse target distribution
* Severe scale challenges
* Highly sensitive to localization errors

Used for:

* ESRGAN/SwinIR joint training
* NWD-based localization
* Tiny-object benchmarking
* IoU vs NWD comparison

---

# Core Pipeline

```text
VisDrone / AI-TOD
        ↓
Tiny Vehicle Filtering
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
RT-DETR-L + NWD Loss
        ↓
Tiny Object Detection
```

---

# Key Components

## RT-DETR-L

RT-DETR-L was used as the primary detection architecture because of:

* End-to-end transformer detection
* Anchor-free localization
* Global feature modeling
* Strong adaptability for small-object detection
* Real-time inference capability

---

## ESRGAN

Enhanced Super-Resolution GAN used for:

* 2× Super-Resolution
* 4× Super-Resolution
* Joint SR-detection training

### Strengths

* Sharp texture generation
* High perceptual realism
* Strong visual enhancement

### Limitations

* Hallucinated textures at high magnification
* Semantic inconsistencies for ultra-tiny targets
* Classification instability during aggressive SR

---

## SwinIR

Transformer-based super-resolution architecture explored as an alternative to ESRGAN.

### Motivation

SwinIR was investigated because it:

* Preserves semantic structure more effectively
* Uses self-attention for global feature consistency
* Reduces hallucinated textures
* Produces more stable RT-DETR feature representations

### Expected Advantages

* Improved precision
* Better semantic consistency
* More stable 4× SR training
* Improved AP_small performance

---

## SAHI (Slicing Aided Hyper Inference)

SAHI was integrated to improve tiny-object detection performance.

### Why SAHI?

Tiny objects become extremely difficult to detect in full-resolution aerial images.

SAHI improves detection by:

```text
Large Image
     ↓
Overlapping Slices
     ↓
Detection on Individual Tiles
     ↓
Prediction Merging
```

Benefits:

* Enlarges tiny objects relative to detector input
* Improves recall significantly
* Enhances small-object localization

---

# NWD Loss (Normalized Wasserstein Distance)

One of the primary research contributions of this project is the comparison between IoU-based localization and NWD-based localization.

---

## Problem with IoU for Tiny Objects

For ultra-small objects:

* A 1–2 pixel shift can drastically reduce IoU
* Tiny localization errors create unstable gradients
* Learning becomes inconsistent

Example:

```text
Ground Truth Box: 6×6
Prediction Shifted by 2 Pixels
→ IoU collapses rapidly
```

---

## NWD Solution

NWD models bounding boxes as Gaussian distributions instead of rectangles.

Bounding Box:

```text
(cx, cy, w, h)
```

becomes:

```text
N(μ, Σ)
```

where:

```text
Σ = [[w²/12, 0],
     [0, h²/12]]
```

---

## Advantages of NWD

Compared to IoU, NWD provides:

* Smoother gradients
* Stable localization learning
* Robustness to tiny pixel shifts
* Better tiny-object alignment
* Improved optimization for aerial targets

Especially beneficial for:

* AI-TOD
* VisDrone tiny vehicles
* High-altitude ISR scenarios

---

# Loss Functions

## Pixel Loss

Used to preserve:

* Spatial consistency
* Geometric structure
* Accurate reconstruction

---

## Perceptual Loss

VGG-based perceptual loss used to preserve:

* Semantic structure
* Object recognizability
* Feature-level consistency

---

## Adversarial Loss

GAN-based realism optimization used to:

* Improve visual sharpness
* Enhance perceptual detail

### Limitation

Can introduce hallucinated textures for ultra-small objects.

---

## Combined Joint Training Loss

```text
L_total = L_cls + L_nwd + λL_sr
```

Where:

* L_cls → classification loss
* L_nwd → NWD localization loss
* L_sr → super-resolution reconstruction loss

---

# Joint Training

A major focus of this work was task-driven SR through joint optimization.

Instead of training:

* SR separately
* Detector separately

the pipeline jointly trains:

```text
ESRGAN / SwinIR
        +
RT-DETR-L
```

This enables:

* Detector gradients to influence SR reconstruction
* SR to optimize for detectability rather than visual beauty
* Better localization-aware reconstruction

---

# Experimental Studies

The project includes the following experimental pipelines:

---

## High-Altitude Tiny Object Detection

Using:

* VisDrone2019-DET
* RT-DETR-L
* ESRGAN
* SAHI slicing-assisted inference

---

## High-Altitude Vehicle Detection

Using:

* Filtered VisDrone tiny vehicle dataset
* RT-DETR-L
* ESRGAN
* SAHI inference

---

## NWD vs IoU Comparison

Using:

* AI-TOD
* VisDrone
* RT-DETR-L
* ESRGAN
* Joint training

Evaluation includes:

* Localization stability
* Recall comparison
* Precision comparison
* AP_small analysis

---

## Video-Based Tiny Object Detection

Includes:

* SAHI-assisted inference
* NWD vs IoU comparison
* High-altitude video detection pipeline
* Tiny-object tracking experiments

---

## Cross-Model Benchmarking

Models compared:

* YOLOv12-L
* YOLOv26-L
* RT-DETR-L

With:

* ESRGAN
* SwinIR
* SAHI slicing
* NWD localization

---

# Experimental Observations

## VisDrone 2×

* Stable convergence
* Strong recall improvement
* Better localization consistency
* Reduced false negatives

---

## VisDrone 4×

* Improved tiny-object reconstruction
* Better NWD alignment
* Strong localization performance

---

## AI-TOD 2×

* Most stable training setup
* Best semantic consistency
* Strong tiny-object recall

---

## AI-TOD 4×

Observed:

* Decreasing NWD localization loss
* Increasing classification instability
* Slight increase in final total loss

### Key Insight

Aggressive 4× super-resolution improved localization while simultaneously introducing semantic instability for ultra-tiny aerial targets.

This highlighted an important trade-off between:

* Localization precision
* Semantic consistency

for extreme SR scales.

---

# Visualization and Analysis

The repository includes:

* Loss convergence plots
* NWD trend visualization
* SR reconstruction comparisons
* Joint training summaries
* ESRGAN/SwinIR qualitative outputs
* Detection result visualizations
* Video inference outputs

---

# Technologies Used

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

# Hardware

Experiments were conducted using:

| GPU         | Usage                             |
| ----------- | --------------------------------- |
| NVIDIA T4   | Baseline training                 |
| NVIDIA L4   | Optimized inference               |
| NVIDIA A10G | Large-scale SR + RT-DETR training |

---

# Repository Structure

```text
project/
│
├── datasets/
├── augmented/
├── degraded/
├── sr_outputs/
├── checkpoints/
├── visualizations/
├── notebooks/
├── runs/
├── weights/
└── README.md
```

---

# Key Research Insights

## Tiny-object detection is recall-limited

Most failures occur because:

* tiny targets are missed
* not because they are incorrectly classified

---

## NWD outperforms IoU for tiny-object localization

NWD provides:

* stable gradients
* better localization learning
* improved tiny-object optimization

---

## Task-driven SR is more important than visual SR

Detection models prioritize:

* semantic consistency
* feature stability
* object structure

rather than photorealistic textures.

---

## SwinIR may outperform ESRGAN for AI-TOD

Transformer-based SR models provide:

* reduced hallucination
* better structural consistency
* more stable feature representations

for ultra-small aerial targets.

---

# Future Work

* SwinIR + RT-DETR-L joint training
* Real-ESRGAN integration
* Detection-aware perceptual losses
* Dynamic SR scaling
* Adaptive NWD weighting
* Transformer-based task-driven SR
* AP_small optimization
* Real-world ISR deployment experiments

---

# Citation

```bibtex
@project{tiny_object_sr_rtdetr,
  title={Task-Driven Super-Resolution for Tiny Object Detection using ESRGAN/SwinIR, RT-DETR-L, SAHI, and NWD Loss},
  author={Arnav Deshpande},
  year={2026}
}
```

---

# Author

Arnav Deshpande
Indian Institute of Technology Indore

Research Areas:

* Machine Learning
* Computer Vision
* Tiny Object Detection
* Aerial AI Systems
* Transformer-based Detection
* Super-Resolution Research

---

# Objective

The primary objective of this project is to improve high-altitude tiny-object detection using task-driven super-resolution, transformer-based detection, slicing-assisted inference, and localization-aware optimization for real-world aerial surveillance and ISR applications.
