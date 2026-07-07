+++
title = "Wildfire Spread Modeling via Reaction–Diffusion PDE"
date = "2026-06-30"
slug = "wildfire"
math = true
tags = ["PDE", "simulation", "Bayesian optimization", "remote sensing"]
icon = "fa-solid fa-fire"
accent = "#e8590c"
summary = "Fitting a spatially-heterogeneous reaction–diffusion PDE to infrared satellite video, with physical parameters calibrated by Bayesian optimization."
+++

A data-driven PDE framework for modeling wildfire temperature evolution from **infrared
satellite video**. Physical parameters are calibrated with Bayesian optimization (Optuna) and
predictions are validated against held-out frames.

<!--more-->

<a class="download-btn" href="https://github.com/Farryn1/wildfire-pde-model" target="_blank" rel="noopener"><i class="fa-brands fa-github"></i> View source on GitHub</a>

## Motivation

Predicting how wildfires spread in space and time is critical for evacuation planning and
resource allocation. This project fits a **spatially-heterogeneous reaction–diffusion PDE** to
real infrared video data, recovering physically interpretable parameters for fire and
background regions separately.

## Model

The temperature field $T(x,y,t)$ and fuel field $Y(x,y,t)$ evolve according to:

$$\frac{\partial T}{\partial t} = D \nabla^2 T - \lambda T + Y \cdot r(T)$$

$$\frac{\partial Y}{\partial t} = -\beta \cdot Y \cdot r(T)$$

where the Arrhenius reaction rate is $r(T) = e^{-1/T}$ for $T > 0$, else $0$.

**Two-region parameterization** — fire pixels and background pixels are assigned separate
parameter sets, defined from the *first frame* to avoid data leakage:

| Region     | Parameters |
|------------|-----------|
| Fire       | $\lambda_\text{fire},\ \beta_\text{fire},\ Y_{0,\text{fire}}$ |
| Background | $\lambda_\text{bg},\ \beta_\text{bg},\ Y_{0,\text{bg}}$ |

## Data

The input is an infrared + RGB satellite video (909 frames ≈ 9 hours of real time). Temperature
is recovered from each frame via **colorbar lookup-table (LUT) inversion**, then normalized to
$[0, 1]$ using ambient ($32.61°C$) and peak fire ($95°C$) reference temperatures.

## Optimization

Parameters are calibrated by minimizing a **region-weighted MSE loss** on the final predicted
frame:

$$\mathcal{L} = w_\text{bg} \cdot \text{MSE}_\text{bg} + w_\text{fire} \cdot \text{MSE}_\text{fire}$$

with $w_\text{fire} = 20$ to compensate for the small fraction of fire pixels. Search uses
[Optuna](https://optuna.org/) (TPE sampler, 30 trials by default).

### Adaptive time step

The explicit Euler time step is chosen adaptively for numerical stability:

$$dt = \text{safety} \times \min\!\left(\frac{1}{4D/\Delta x^2 + \lambda_\max},\ \frac{1}{\beta_\max}\right)$$

A sensitivity sweep over `safety` ∈ {0.05, 0.10, 0.20, 0.40} re-fits all parameters at each
value and selects the setting with the lowest final-frame loss.

## Results

The fitted PDE is evaluated on the final frame of the full 909-frame rollout and compared
against an **initial-frame persistence baseline** (predict last = first), measured by MSE on
the fire region and background separately. Diagnostic outputs include a truth / PDE / baseline
side-by-side comparison, MSE-over-time and max-error-over-time curves, and the safety-vs-loss
sensitivity analysis.

## Technical notes

- **Colorbar inversion:** temperature is read from a per-frame colorbar strip via
  nearest-neighbor lookup in RGB space.
- **Valid-region mask:** colorbar and timestamp-overlay strips are excluded from the loss.
- **Numerical stability:** explicit Euler with a CFL-based adaptive step; zero-flux (Neumann)
  boundary conditions.
- **$Y_0$ parameterization:** initial fuel is a two-value field (fire / background); per-pixel
  $Y_0$ fitting is left as future work due to the curse of dimensionality at high parameter
  counts.

*Stack: Python · NumPy · Optuna · OpenCV*
