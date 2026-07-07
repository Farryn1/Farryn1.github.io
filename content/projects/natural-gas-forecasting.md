+++
title = "Henry Hub Natural Gas Price Forecasting"
date = "2026-06-29"
slug = "natural-gas-forecasting"
tags = ["time series", "forecasting", "XGBoost", "GARCH", "LSTM"]
icon = "fa-solid fa-chart-line"
accent = "#0ca678"
summary = "A nine-model benchmark for Henry Hub gas prices — classical time series, GARCH, and ML — under fixed vs. rolling forecasting protocols."
+++

A forecasting benchmark comparing **nine model families** on Henry Hub natural gas spot prices —
spanning classical time series (ARIMA/SARIMA, ARMA–GARCH), exogenous-regressor models
(ARIMAX/SARIMAX), and machine learning (XGBoost, LSTM, Silverkite). Both **fixed multi-step**
and **rolling one-step-ahead** evaluation protocols are implemented.

<!--more-->

<a class="download-btn" href="https://github.com/Farryn1/henry-hub-forecasting" target="_blank" rel="noopener"><i class="fa-brands fa-github"></i> View source on GitHub</a>

## Key findings

| Finding | Detail |
|---------|--------|
| **Protocol dominates model choice** | Rolling 1-step-ahead cuts MAPE ~4× vs. fixed multi-step across every model |
| **Best daily model** | Rolling ARMA(3,0,3)–GARCH(1,1) Student-t: **MAPE 3.36%** (beats naïve 3.49%) |
| **Best weekly model** | XGBoost with lag + exogenous features: **MAPE 4.70%** rolling |
| **LSTM underperforms** | Only ~800 training obs after inner-join; sample-starved vs. model size |
| **Silverkite** | Changepoint regularization too conservative for this series |

## Data

All data is publicly available (EIA, FRED, NOAA CPC), with download scripts provided.

| Series | Source | Frequency | Coverage | Obs. |
|--------|--------|-----------|----------|------|
| Henry Hub spot price | EIA | Daily | 1997–2026 | 7,348 |
| Henry Hub spot price | EIA | Weekly | 1997–2026 | 1,526 |
| U.S. Working Gas Storage | EIA | Weekly | 2010–2026 | 849 |
| WTI crude oil | FRED | Weekly | 1986–2026 | 2,154 |
| Heating Degree Days (HDD) | NOAA CPC | Weekly | 1997–2026 | 1,480 |

Exogenous regressors are available from 2010, so exogenous models are restricted to the
2010–2026 inner-joined window.

{{< figure src="/images/projects/gas_eda.png" alt="Exploratory data analysis" caption="Henry Hub spot price series and exploratory analysis." >}}

{{< figure src="/images/projects/gas_correlation.png" alt="Correlation analysis" caption="Correlation between price and exogenous regressors (storage, WTI, HDD)." >}}

## Models

- **Benchmarks:** Naïve (random walk), Drift, Seasonal Naïve (S = 52).
- **Classical (`statsmodels`):** ARIMA, SARIMA, ARIMAX, SARIMAX with WTI / storage / HDD
  regressors (lagged one week).
- **ARMA–GARCH (`arch`):** two-step on log returns — ARMA conditional mean, then GARCH(1,1)
  with Student-*t* innovations; price levels recovered by exponentiation with GARCH-widened
  intervals.
- **XGBoost:** a 23-dim feature matrix (price & log-return lags {1,2,3,4,8,13,52}, lag-1
  storage/WTI/HDD, cyclic calendar features); 200 trees, depth 3, lr 0.05.
- **LSTM (PyTorch):** 2 × LSTM(64, dropout 0.2) → FF 64→32→1 over 12-week sliding sequences;
  Adam, MSE, early stopping.
- **Silverkite (Greykite):** changepoint detection + Fourier seasonality + ridge-penalized
  exogenous regressors.

{{< figure src="/images/projects/gas_xgb_importance.png" alt="XGBoost feature importance" caption="XGBoost feature importances — price lags and exogenous regressors." >}}

## Results

{{< figure src="/images/projects/gas_mape_comparison.png" alt="MAPE comparison" caption="Forecast MAPE across all model families." >}}

{{< figure src="/images/projects/gas_rolling_forecasts.png" alt="Rolling forecasts" caption="Rolling one-step-ahead forecasts against realized prices." >}}

| Model | Fixed MAPE | Rolling MAPE |
|-------|:----------:|:------------:|
| Naïve | 3.49% | 3.49% |
| Drift | 3.46% | — |
| Seasonal Naïve | 34.67% | — |
| ARIMA(2,0,2) W | 29.87% | 9.52% |
| SARIMA W | 26.55% | 9.12% |
| ARIMAX W | 40.82% | 13.54% |
| SARIMAX W | 26.23% | 11.50% |
| **ARMA–GARCH W** | 15.54% | 5.15% |
| **XGBoost W** | **5.29%** | **4.70%** |
| LSTM W | 12.90% | 12.90% |
| Silverkite W | 36.46% | 23.78% |
| ARIMA(4,0,4) D | 18.36% | 3.70% |
| SARIMA D | 26.79% | 4.00% |
| **ARMA–GARCH D** | 10.39% | **3.36%** |

The headline result is that **evaluation protocol matters more than model family**: rolling
one-step-ahead re-estimation cuts error roughly fourfold across the board, and a well-tuned
ARMA–GARCH or XGBoost is needed to beat the naïve random walk.

*Stack: Python · statsmodels · arch · XGBoost · PyTorch · Greykite · Pandas*
