+++
title = "Research"
slug = "research"
+++

My research sits at the intersection of **optimization, statistics, and machine learning**,
with a consistent theme: using data to make better operational decisions. Below is a summary
of my ongoing and recent academic work. Several of these directions are also released as
self-contained, runnable code — see the [Projects](/projects/) page.

---

## Wildfire Spread Modeling from Satellite Imagery

*Apr 2026 – Present*

- Developed a **pixel-to-temperature conversion pipeline** for GOES-17 infrared video
  (Channel 07), replacing a flawed greyscale method with a **colormap-inversion lookup-table
  (LUT)** pipeline.
- Implemented and debugged a **PDE-based numerical simulation** of fire spread, diagnosing and
  correcting a time-scale mismatch that had caused the physics-based model to underperform a
  naïve baseline.

→ Code & write-up: [Wildfire Spread Modeling via Reaction-Diffusion PDE](/projects/wildfire/)

---

## Modeling and Forecasting of Henry Hub Natural Gas Spot Prices

*Apr 2026 – Jun 2026*

- Developed and compared **time-series and machine-learning models** (ARIMA/SARIMA,
  ARIMAX/SARIMAX, ARMA–GARCH, XGBoost, LSTM, Silverkite) for short-term forecasting of Henry
  Hub natural gas spot prices, incorporating **U.S. working-gas storage** and **heating degree
  days (HDD)** as exogenous variables.
- Analyzed dynamic price behaviors driven by supply–demand fluctuations, weather shocks, and
  market volatility, and evaluated model performance under **fixed multi-step** and **rolling
  one-step-ahead** forecasting frameworks.

→ Code & write-up: [Henry Hub Natural Gas Price Forecasting](/projects/natural-gas-forecasting/)

---

## Expectation–Experience Gap Modeling for Hotel Reviews

*Jan 2026 – Jun 2026 · Advised by Prof. Xinxin Wang*

- Formulated an **expectation–experience gap modeling framework** for hotel reviews,
  decomposing user feedback into **aspect coverage, sentiment polarity, and helpfulness
  weights** to prioritize service dimensions.
- Designed a **multimodal modeling pipeline** to extract structured representations from
  platform promotional content (images / text) and customer reviews, using **Transformer-based
  encoders** to project content into a unified *K*-aspect space (e.g., room, service,
  breakfast, location).
- Completed system design and feature specification; currently implementing aspect-level
  modeling with planned optimization for gap estimation and ranking.
