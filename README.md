# dengue-outbreak-forecasting

Spatio-Temporal Graph Neural Network (ST-GNN) research for multi-horizon,
district-level dengue outbreak forecasting in Sri Lanka, benchmarked
against SARIMAX, XGBoost, and LSTM baselines.

## Overview

This repository contains the research pipeline and models for forecasting
weekly dengue case counts across Sri Lanka's 25 administrative districts
at **2-week (t+2)** and **4-week (t+4)** horizons. It combines 19 years
(2007–2026) of Weekly Epidemiological Report (WER) case data, NASA POWER
meteorological time series, and district-level spatial topology to
develop and evaluate a Spatio-Temporal Graph Neural Network against
classical statistical and deep learning baselines, aiming to capture
both cross-district transmission spillover and lagged, climate-driven
outbreak dynamics for earlier, more actionable public health early
warning.

**Core research question:** Can modeling Sri Lanka's 25 districts as an
interconnected graph, fusing weekly case counts with meteorological
time series, outperform single-series statistical and tabular ML
baselines in multi-horizon dengue forecasting?

## Data

| Component | Description | Source |
|---|---|---|
| Epidemiological data | Weekly notified dengue cases, all 25 districts, 2007–2026 | [Weekly Epidemiological Reports](https://www.epid.gov.lk), Epidemiology Unit, Ministry of Health, Sri Lanka |
| Meteorological data | Weekly mean/min/max temperature, precipitation, relative humidity, wind speed per district | [NASA POWER API](https://power.larc.nasa.gov) |
| Spatial graph topology | District adjacency matrix and inter-district centroid distances | [OCHA COD-AB Sri Lanka](https://data.humdata.org/dataset/cod-ab-lka) (Survey Department of Sri Lanka) |

**Data notes:**
- Kalmunai (historically reported separately in WER data, not one of the
  25 official administrative districts) is merged into Ampara.
- 2026 is a partial year and is excluded from model training/evaluation.
- Case and weather data join cleanly on `(year, week, district)` with no
  missing weather rows across the full panel.

## Baseline results

All models trained on 2007–2022, validated on 2023, tested on 2024–2025,
across all 25 districts.

| Horizon | Model | MAE | RMSE | Pearson r |
|---|---|---|---|---|
| t+2 | **SARIMAX (per-district)** | **11.42** | **21.71** | 0.942 |
| t+2 | LSTM (global, tuned) | 11.95 | 24.09 | 0.924 |
| t+2 | XGBoost (global, tuned) | 12.49 | 25.61 | 0.917 |
| t+2 | Naive persistence | 13.88 | 30.27 | 0.895 |
| t+4 | **LSTM (global, tuned)** | **14.87** | 29.45 | 0.877 |
| t+4 | SARIMAX (per-district) | 15.09 | 31.71 | 0.872 |
| t+4 | XGBoost (global, tuned) | 15.42 | 30.72 | 0.872 |
| t+4 | Naive persistence | 17.25 | 39.06 | 0.820 |

All baselines beat naive persistence at both horizons. SARIMAX is
strongest at t+2; LSTM overtakes it at t+4, motivating the proposed
spatio-temporal graph extension for the longer, clinically more useful
early-warning horizon.

## Method (proposed)

The ST-GNN architecture separates spatial and temporal modeling:

```
Input: [Batch, Lookback (8 weeks), 25 Districts, Features]
   │
   ├─► Temporal Encoder ─► 1D-TCN / Mamba (selective state-space)
   │
   ├─► Spatial Encoder  ─► Graph Convolutional Network (ChebConv/GCNConv)
   │
   └─► Prediction Head  ─► forecasts for t+2 and t+4
```

## Citation

If you use this repository or data pipeline, please cite the underlying
data sources directly:

- Epidemiology Unit, Ministry of Health, Sri Lanka - Weekly
  Epidemiological Report series (case data)
- NASA POWER Project - meteorological data
- OCHA COD-AB Sri Lanka / Survey Department of Sri Lanka - administrative
  boundaries

## Authors

Sonath Kirindage, Vihanga Nimsara, Sakindu Rajapaksa, Kavyanga
Hathurusinghe, Lahiru Dilshan, Sandareka Wickramanayake


