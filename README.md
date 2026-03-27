# Monocular-distance-estimation
# Physics-Based Monocular Distance Estimation
### Using Pinhole Camera Geometry and Probabilistic Error Modeling

> **Academic Project** | Probability and Random Processes (PRP) | Delhi Technological University  
> **Author:** Aman Dagar | **Supervisor:** Prof. Rajeev Kumar Kapoor

---

## Overview

This project presents a **fully physics-grounded, training-free monocular distance estimation system** developed from first principles using classical pinhole camera geometry, analytical uncertainty propagation, and Gaussian probabilistic modeling.

Unlike machine learning or deep learning approaches that operate as black boxes and require large annotated datasets, this system derives a **closed-form analytical solution** directly from the physical laws governing camera optics. The model is transparent, interpretable, generalizable, and mathematically verifiable at every stage — properties that data-driven models fundamentally cannot guarantee.

The project was developed iteratively across two architectural versions (Mark-1 and Mark-2), culminating in an engineering-grade solution that bridges classical physics, geometric optics, and applied probability theory.

---

## Table of Contents

- [Problem Statement](#problem-statement)
- [System Architecture](#system-architecture)
- [Physical Model Derivation](#physical-model-derivation)
- [Probabilistic Error Modeling](#probabilistic-error-modeling)
- [Experimental Setup](#experimental-setup)
- [Model Performance](#model-performance)
- [Implementation](#implementation)
- [Results](#results)
- [Repository Structure](#repository-structure)
- [Resources & Links](#resources--links)
- [Academic Declaration](#academic-declaration)

---

## Problem Statement

The determination of real-world distance from a single 2D image — known as monocular depth/distance estimation — is a fundamental and open problem in computer vision. Existing solutions predominantly rely on deep neural networks trained on massive datasets, making them computationally expensive, opaque in decision-making, and weakly generalizable across different camera configurations.

This project establishes and validates the proposition that **for objects of known real-world height, accurate and probabilistically bounded distance estimation can be achieved entirely through closed-form physical computation**, without any training, gradient optimization, or learned parameters whatsoever.

---

## System Architecture

The complete computational pipeline is structured as follows:

```
Image Capture
    ↓
Bounding Box Annotation (Manual / Automated)
    ↓
Pixel Height Extraction (h_px)
    ↓
Camera Parameter Computation (f_px from hardware specs)
    ↓
Model Constant Derivation (k = f_px × H)
    ↓
Distance Estimation (D = k / h_px)
    ↓
Uncertainty Propagation (σ_D = k / h_px² × σ_px)
    ↓
Gaussian PDF Construction → Probabilistic Distance Estimate
```

---

## Physical Model Derivation

### 5.1 — Pinhole Camera Principle

The model is grounded in the **similar triangles relationship** of the pinhole camera projection:

$$\frac{h}{f} = \frac{H}{D}$$

Rearranging to isolate pixel height:

$$h = \frac{f \cdot H}{D} = \frac{k}{D}$$

Where the **model constant** `k` is defined as:

$$k = f_{px} \cdot H$$

Inverting to yield the **core distance estimation equation**:

$$D = \frac{k}{h_{px}}$$

### 5.2 — Focal Length Conversion (mm → pixels)

Camera hardware specifications express focal length in millimeters. Since all image measurements are in pixels, the focal length must be converted as:

$$f_{px} = \left(\frac{f_{mm}}{sensor\_width_{mm}}\right) \times image\_width_{px}$$

Substituting the experimental camera parameters:

$$f_{px} = \left(\frac{5.23}{6.4}\right) \times 4624 \approx \mathbf{3778 \ px}$$

Computing the model constant with object height H = 1.05 m:

$$k = 3778 \times 1.05 \approx \mathbf{3968}$$

### Final Analytical Distance Model

$$\boxed{D = \frac{3968}{h_{px}}}$$

This single equation enables **direct, training-free, real-time distance prediction** from observed pixel height alone.

---

## Probabilistic Error Modeling

### Measurement Noise Characterization

Manual bounding-box annotation introduces stochastic pixel-level noise, experimentally characterized as:

$$\sigma_{px} \approx 55 \ \text{pixels}$$

### Distance Uncertainty Propagation

Applying first-order error propagation to the distance model:

$$\sigma_D = \left|\frac{dD}{dh}\right| \cdot \sigma_{px} = \frac{k}{h_{px}^2} \cdot \sigma_{px}$$

**Critical observation:** Distance uncertainty grows **quadratically** with distance — a direct consequence of the inverse relationship between pixel height and distance. This has significant practical implications for long-range estimation systems.

### Gaussian Probabilistic Distance Distribution

Given the superposition of multiple independent noise sources (annotation imprecision, micro hand-movement during capture, lens aberrations), the Central Limit Theorem guarantees that the aggregate measurement error converges to a **Gaussian distribution**:

$$P(D) \sim \mathcal{N}(\mu = D_{pred}, \ \sigma = \sigma_D)$$

This probabilistic formulation enables:
- Confidence interval construction (e.g., 95% CI = D ± 1.96σ_D)
- Likelihood-based decision making
- Statistically rigorous uncertainty quantification

### Worked Example

For an observed pixel height of h = 800 px:

$$D = \frac{3968}{800} = 4.96 \ \text{m}$$
$$\sigma_D = \frac{3968}{800^2} \times 55 \approx 0.34 \ \text{m}$$
$$\therefore \ D = 4.96 \pm 0.34 \ \text{meters} \ (1\sigma)$$

---

## Experimental Setup

| Parameter | Value |
|---|---|
| Camera | Samsung Galaxy M31 |
| Sensor Resolution | 64 MP (4624 × 3468 px) |
| Focal Length | 5.23 mm |
| Aperture | f/1.8 |
| Sensor Width | 6.4 mm |
| Object Height (H) | 1.05 meters |
| Measurement Distances | 1, 2, 4, 8, 12, 20 meters |
| Images per Distance | 3 (center, left, right viewpoints) |
| Total Dataset Size | 18 annotated samples |

Images were captured at fixed known distances under controlled conditions. Three viewpoints per distance were used to incorporate real-world viewpoint variation and quantify annotation noise empirically.

---

## Model Performance

### Sample Predictions

| Pixel Height (px) | Estimated Distance (m) |
|---|---|
| 2200 | 1.80 |
| 1800 | 2.20 |
| 1200 | 3.31 |
| 800 | 4.96 |
| 400 | 9.92 |

### Key Validated Observations

- Pixel height exhibits a **strong, clean inverse relationship** with distance, fully consistent with the pinhole camera model
- The analytical model curve aligns closely with experimentally observed data points
- Distance uncertainty scales quadratically — prediction confidence is highest at close range and degrades predictably at greater distances
- The Gaussian PDF produces statistically valid confidence bounds across all tested distances

---

## Model Development: Mark-1 → Mark-2

### Mark-1 — Data-Driven Regression (Baseline)

$$h = a \cdot \frac{1}{D} + b$$

| Property | Status |
|---|---|
| Requires training data | ✅ Yes |
| Physically interpretable | ❌ No |
| Generalizable across cameras | ❌ No |
| Uncertainty modeling | ❌ Not supported |

### Mark-2 — Physics-Based Analytical Model (Final)

$$D = \frac{k}{h_{px}}, \quad k = f_{px} \cdot H$$

| Property | Status |
|---|---|
| Requires training data | ✅ None required |
| Physically interpretable | ✅ Fully |
| Generalizable across cameras | ✅ Yes — recalibrate k only |
| Uncertainty modeling | ✅ First-order propagation + Gaussian PDF |

Mark-2 is not merely an improvement over Mark-1 — it is an architectural shift from empirical curve-fitting to a **first-principles engineering solution**.

---

## Implementation

### Dependencies

```bash
pip install numpy pandas matplotlib scipy
```

### Core Functions

```python
import numpy as np
from scipy.stats import norm

# Camera & model constants
f_mm = 5.23
sensor_width_mm = 6.4
image_width_px = 4624
H = 1.05  # Real object height in meters

f_px = (f_mm / sensor_width_mm) * image_width_px  # ≈ 3778 px
k = f_px * H  # ≈ 3968

def predict_distance(pixel_height: float) -> float:
    """Returns estimated distance in meters from observed pixel height."""
    return k / pixel_height

def distance_uncertainty(pixel_height: float, sigma_px: float = 55) -> float:
    """Returns 1-sigma distance uncertainty via first-order error propagation."""
    return abs(k / pixel_height**2) * sigma_px

def gaussian_pdf(pixel_height: float, sigma_px: float = 55):
    """Returns x-axis values and Gaussian PDF of distance estimate."""
    d_pred = predict_distance(pixel_height)
    sigma_d = distance_uncertainty(pixel_height, sigma_px)
    x = np.linspace(d_pred - 4*sigma_d, d_pred + 4*sigma_d, 400)
    pdf = norm.pdf(x, d_pred, sigma_d)
    return x, pdf, d_pred, sigma_d
```

---

## Results

The Observed vs. Physics Model plot demonstrates strong agreement between analytically predicted and experimentally measured pixel heights across all six distance levels. The Gaussian PDF plot for h = 800 px confirms a well-formed, narrow probability distribution centred at D = 4.96 m with σ = 0.34 m — indicating high-confidence distance estimation in the sub-10m range.

---

## Applications

This framework is directly applicable, without modification, to:

- **Autonomous vehicle** proximity and obstacle distance sensing
- **Robotics navigation** and spatial awareness systems
- **Drone altitude estimation** from ground-reference objects
- **CCTV surveillance analytics** for crowd density and distance measurement
- **Augmented reality** depth layer integration
- **Computer vision geometry** research and calibration benchmarking

---

## Repository Structure

```
PRP_Project/
│
├── README.md                     ← This document
├── Mark1_Regression.ipynb        ← Mark-1: Data-driven baseline model
├── Mark2_Physics.ipynb           ← Mark-2: Physics-based analytical model
│
├── data/
│   ├── metadata.csv              ← Image filenames and capture distances
│   └── annotations.csv           ← Bounding box pixel height annotations
│
└── images/                       ← Raw captured image dataset
```

---

## Resources & Links

| Resource | Link |
|---|---|
| Mark-1 Google Colab Notebook | [Open in Colab](https://colab.research.google.com/drive/1xxHA_BfoeySaeE0QvT2qg8hQhJb-wrrF?usp=sharing) |
| Mark-2 Notebook | PRP Mark 2 .ipynb (included in repo) |
| CSV Dataset (metadata + annotations) | [Google Drive](https://drive.google.com/drive/folders/13HF8wDYDyIoTmhDnNi34fqQd8h1Ln0Os?usp=sharing) |
| Image Dataset | [Google Drive](https://drive.google.com/drive/folders/1CR6XhUEtnG5ELlUSVwxsblewI0mAybuf?usp=sharing) |

All experimental data, code, and outputs are made publicly available to ensure full reproducibility and independent verification of results.

---

## Academic Declaration

This project was conceived, designed, and executed in its entirety by **Aman Dagar** as an original academic submission for the course **Probability and Random Processes (PRP)** at **Delhi Technological University (DTU)**, under the guidance of **Prof. Rajeev Kumar Kapoor**.

All referenced methodologies, external frameworks, and experimental observations have been duly acknowledged. No portion of this work has been submitted elsewhere for academic credit.

---

*Delhi Technological University | Shahbad Daulatpur, Main Bawana Road, Delhi – 110042*
