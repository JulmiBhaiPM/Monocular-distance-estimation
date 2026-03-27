# Physics-Based Monocular Distance Estimation
### Pinhole Camera Geometry · Probabilistic Error Modeling · Zero Training Required

> **B.Tech Engineering Project** | Delhi Technological University  
> **Domain:** Computer Vision · Applied Probability · Classical Optics

---

## What This Project Does — In One Line

Given a single photograph, this system **computes the real-world distance of an object directly from its pixel height** — using nothing but physical camera geometry and mathematics. No neural network. No training data. No black box.

---

## Why This Is Different

The dominant approach to monocular distance estimation today is deep learning: train a massive neural network on millions of labelled images, and hope it generalizes. That approach works numerically but fails on three critical counts — it is **physically uninterpretable**, **camera-dependent**, and **statistically opaque**.

This project takes the opposite position.

Every parameter in this model has a physical meaning. Every prediction comes with a mathematically derived uncertainty bound. Every result is reproducible from a single equation. The system requires no training, no GPU, and no dataset beyond 18 manually captured images. It runs on the laws of optics — which do not overfit.

This is not a course assignment uploaded to a repository. This is an independently designed, documented, and validated engineering system that demonstrates what rigorous first-principles thinking can achieve in a domain otherwise dominated by brute-force computation.

---

## Table of Contents

- [Core Principle](#core-principle)
- [Physical Model Derivation](#physical-model-derivation)
- [Probabilistic Error Modeling](#probabilistic-error-modeling)
- [Experimental Setup](#experimental-setup)
- [Model Evolution: Mark-1 → Mark-2](#model-evolution-mark-1--mark-2)
- [Sample Predictions](#sample-predictions)
- [Implementation](#implementation)
- [Applications](#applications)
- [Repository Structure](#repository-structure)
- [Resources & Links](#resources--links)
- [Author](#author)

---

## Core Principle

The pinhole camera model establishes a precise geometric relationship between an object's real-world size, its apparent size in pixels, and its distance from the camera. This relationship — grounded in the law of similar triangles — yields a closed-form, invertible equation:

$$\frac{h}{f} = \frac{H}{D} \implies D = \frac{f_{px} \cdot H}{h_{px}}$$

Where:
- `D` = distance to object (meters) — **what we want**
- `H` = known real-world object height (meters) — **fixed**
- `h_px` = observed pixel height in image — **measured**
- `f_px` = focal length in pixels — **derived from hardware specs**

This is the entirety of the model. One equation. Fully interpretable. Infinitely auditable.

---

## Physical Model Derivation

### Step 1 — Focal Length Conversion (mm → pixels)

Camera manufacturers report focal length in millimeters. All image measurements are in pixels. The conversion is:

$$f_{px} = \left(\frac{f_{mm}}{sensor\_width_{mm}}\right) \times image\_width_{px}$$

Substituting hardware specifications of the Samsung Galaxy M31:

$$f_{px} = \left(\frac{5.23}{6.4}\right) \times 4624 \approx \mathbf{3778 \ px}$$

### Step 2 — Model Constant

With real object height H = 1.05 m:

$$k = f_{px} \cdot H = 3778 \times 1.05 \approx \mathbf{3968}$$

### Step 3 — Final Distance Equation

$$\boxed{D = \frac{3968}{h_{px}}}$$

This single expression constitutes the complete, deployment-ready distance estimation model. No weights. No hyperparameters. No training loop.

---

## Probabilistic Error Modeling

A physics model is only as credible as its uncertainty quantification. This project does not simply output a distance — it outputs a **probability distribution over possible distances**, rigorously derived from first-order error propagation.

### Noise Characterisation

Manual bounding-box annotation introduces stochastic pixel-level noise, experimentally characterised as:

$$\sigma_{px} \approx 55 \ \text{pixels}$$

### First-Order Uncertainty Propagation

Differentiating `D = k / h` with respect to `h`:

$$\sigma_D = \left|\frac{dD}{dh}\right| \cdot \sigma_{px} = \frac{k}{h_{px}^2} \cdot \sigma_{px}$$

**Critical finding:** Distance uncertainty scales **quadratically** with distance — not linearly. This means the model is highly precise at close range and systematically degrades at greater distances. This is not a flaw; it is a mathematically necessary consequence of the inverse projection geometry, and it is fully quantified.

### Gaussian Distance Distribution

By the Central Limit Theorem, the superposition of independent noise sources (annotation imprecision, micro hand-motion during capture, lens aberration) converges to a Gaussian:

$$P(D) \sim \mathcal{N}(\mu = D_{pred}, \ \sigma = \sigma_D)$$

This enables statistically rigorous confidence intervals:
- **68% confidence:** D ± σ_D
- **95% confidence:** D ± 1.96σ_D
- **99.7% confidence:** D ± 3σ_D

### Worked Example

For h = 800 px:

$$D = \frac{3968}{800} = 4.96 \ \text{m}$$
$$\sigma_D = \frac{3968}{800^2} \times 55 \approx 0.34 \ \text{m}$$

$$\therefore \ D = \mathbf{4.96 \pm 0.34 \ meters} \ \text{at } 1\sigma$$

---

## Experimental Setup

| Parameter | Specification |
|---|---|
| Camera | Samsung Galaxy M31 |
| Sensor Resolution | 64 MP — 4624 × 3468 px |
| Focal Length | 5.23 mm |
| Aperture | f/1.8 |
| Sensor Width | 6.4 mm |
| Object Height (H) | 1.05 meters |
| Capture Distances | 1, 2, 4, 8, 12, 20 meters |
| Images per Distance | 3 — center, left, right viewpoints |
| Total Annotated Samples | 18 |

Three viewpoints per distance were deliberately used to incorporate real-world viewpoint variation and empirically characterise annotation noise — rather than assuming it.

---

## Model Evolution: Mark-1 → Mark-2

### Mark-1 — Data-Driven Regression (Baseline)

$$h = a \cdot \frac{1}{D} + b$$

The baseline model fitted a regression curve to experimental data. It produced numerically reasonable outputs but was fundamentally limited:

| Criterion | Mark-1 |
|---|---|
| Requires training data | ✅ Yes |
| Parameters physically meaningful | ❌ No |
| Generalises to new cameras | ❌ No |
| Uncertainty quantification | ❌ None |
| Explainable predictions | ❌ No |

### Mark-2 — Physics-Based Analytical Model (Final)

$$D = \frac{k}{h_{px}}, \quad k = f_{px} \cdot H$$

| Criterion | Mark-2 |
|---|---|
| Requires training data | ✅ Zero |
| Parameters physically meaningful | ✅ Fully |
| Generalises to new cameras | ✅ Yes — recompute k only |
| Uncertainty quantification | ✅ First-order propagation |
| Explainable predictions | ✅ Every step auditable |

Mark-2 is not an incremental improvement. It is a **complete architectural shift** from empirical curve-fitting to a physics-grounded analytical solution — the difference between measuring shadows and understanding light.

---

## Sample Predictions

| Pixel Height (px) | Estimated Distance (m) |
|---|---|
| 2200 | 1.80 |
| 1800 | 2.20 |
| 1200 | 3.31 |
| 800 | 4.96 |
| 400 | 9.92 |

All predictions exhibit the expected strong inverse proportionality relationship, fully consistent with the physical model. The analytical curve aligns closely with experimentally observed data across all six tested distances.

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
import matplotlib.pyplot as plt

# ── Camera & model constants ──────────────────────────────────────────────────
F_MM        = 5.23
SENSOR_W_MM = 6.4
IMAGE_W_PX  = 4624
H_REAL      = 1.05   # real object height in meters
SIGMA_PX    = 55     # experimentally observed annotation noise

f_px = (F_MM / SENSOR_W_MM) * IMAGE_W_PX   # ≈ 3778 px
k    = f_px * H_REAL                         # ≈ 3968

# ── Core model ────────────────────────────────────────────────────────────────
def predict_distance(pixel_height: float) -> float:
    """Closed-form distance estimate from observed pixel height (meters)."""
    return k / pixel_height

def distance_uncertainty(pixel_height: float, sigma_px: float = SIGMA_PX) -> float:
    """1-sigma distance uncertainty via first-order error propagation (meters)."""
    return abs(k / pixel_height**2) * sigma_px

def gaussian_distance_pdf(pixel_height: float, sigma_px: float = SIGMA_PX):
    """Returns x-axis, PDF values, mean, and std of the distance distribution."""
    mu    = predict_distance(pixel_height)
    sigma = distance_uncertainty(pixel_height, sigma_px)
    x     = np.linspace(mu - 4*sigma, mu + 4*sigma, 400)
    pdf   = norm.pdf(x, mu, sigma)
    return x, pdf, mu, sigma

# ── Example usage ─────────────────────────────────────────────────────────────
h_test = 800
x, pdf, mu, sigma = gaussian_distance_pdf(h_test)

print(f"Pixel Height : {h_test} px")
print(f"Distance     : {mu:.2f} +/- {sigma:.2f} m  (1 sigma)")
print(f"95% CI       : [{mu - 1.96*sigma:.2f}, {mu + 1.96*sigma:.2f}] m")

plt.plot(x, pdf)
plt.xlabel("Distance (m)")
plt.ylabel("Probability Density")
plt.title(f"Gaussian PDF — Distance Estimate for h = {h_test} px")
plt.grid(alpha=0.3)
plt.tight_layout()
plt.show()
```

---

## Applications

This framework applies directly, without modification, to any domain requiring training-free, physics-grounded single-camera distance estimation:

- **Autonomous vehicles** — real-time obstacle proximity detection
- **Robotics** — spatial navigation and environment mapping
- **Drone systems** — altitude estimation from ground-reference objects
- **CCTV analytics** — crowd density and distance measurement
- **Augmented reality** — geometry-accurate depth layer integration
- **Computer vision research** — camera calibration and geometric benchmarking

---

## Repository Structure

```
Monocular-distance-estimation/
│
├── README.md                  ← This document
├── PRP_Mark_1_.ipynb          ← Mark-1: Data-driven regression baseline
├── PRP_Mark_2_.ipynb          ← Mark-2: Physics-based analytical model
│
├── annotations.csv            ← Bounding box pixel height annotations (18 samples)
├── metadata.csv               ← Image filenames and capture distances
│
└── LICENSE
```

---

## Resources & Links

| Resource | Link |
|---|---|
| Mark-1 — Google Colab Notebook | [Open in Colab](https://colab.research.google.com/drive/1xxHA_BfoeySaeE0QvT2qg8hQhJb-wrrF?usp=sharing) |
| Mark-2 — Google Colab Notebook | [Open in Colab](https://colab.research.google.com/drive/1w0B0_JVOhXmFtKVtpuMTwVruRaklA6Bq?usp=sharing) |
| CSV Dataset (metadata + annotations) | [Google Drive](https://drive.google.com/drive/folders/13HF8wDYDyIoTmhDnNi34fqQd8h1Ln0Os?usp=sharing) |
| Image Dataset (raw captures) | [Google Drive](https://drive.google.com/drive/folders/1CR6XhUEtnG5ELlUSVwxsblewI0mAybuf?usp=sharing) |

All experimental data, source code, and outputs are publicly available to ensure complete reproducibility and independent verification of all reported results.

---

## Author

**Aman Dagar**  
B.Tech Student — Delhi Technological University  
[GitHub](https://github.com/JulmiBhaiPM) · [LinkedIn](https://www.linkedin.com/in/aman-d-a664ab310/) · appamandagar753@gmail.com

Originally developed as part of academic coursework in Probability and Random Processes at DTU, this project has been independently extended and documented as part of my personal engineering portfolio.

---

*If you find this project useful, a ⭐ on the repository is appreciated.*
