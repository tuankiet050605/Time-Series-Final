# Daily Temperature Forecasting
**Time Series Analysis using SARIMA, SARIMAX & XGBoost — Mumbai, India (2016–2020)**

---

## Overview

This project forecasts daily temperature in Mumbai, India using 1,781 daily
meteorological observations from 2016 to 2020. Three models are built and
compared: a univariate SARIMA, a multivariate SARIMAX with Granger-selected
exogenous variables, and an XGBoost gradient-boosting model.

The key methodological contribution is the use of **Granger causality testing**
to select exogenous variables for SARIMAX — identifying wind direction,
sea-level pressure, humidity, and dew point as statistically significant
predictors — rather than relying on Pearson correlation alone.

**Main result:** SARIMAX achieves the highest forecast accuracy on the test set
(R² = 0.976, RMSE = 0.313°C, MAPE = 0.83%), outperforming both XGBoost
(R² = 0.938) and SARIMA (R² = 0.836).

---

## Repository Structure

```
📦 temperature-forecasting/
 ├── 📓 01_data_understanding.ipynb
 ├── 📓 02_eda.ipynb
 ├── 📓 03_preprocess_fe.ipynb
 ├── 📓 04_modeling.ipynb
 ├── 📄 rainfall.csv                  ← raw dataset
 ├── 📄 rainfall_fe.csv               ← processed dataset (output of 03)
 ├── 📄 requirements.txt
 └── 📄 README.md
```

> **Run notebooks in order** — each notebook depends on outputs from the previous one.

---

## Dataset

| Attribute | Detail |
|---|---|
| Source | Daily meteorological observations, Mumbai, India |
| Period | 2016-01-01 → 2020-11-15 |
| Observations | 1,781 days |
| Target | `temp` — Daily mean temperature (°C) |
| Features | `dew`, `humidity`, `sealevelpressure`, `winddir`, `solarradiation`, `windspeed`, `preciptype` |

---

## Workflow

### 01 — Data Understanding
- Variable types, data dictionary, basic statistics
- Data quality checks (duplicates, missing values, time continuity)
- Quick overview plots

### 02 — Exploratory Data Analysis
- Distribution analysis (histogram, boxplot, Q-Q plot)
- Time series decomposition with rolling mean and trend (+0.067°C/year)
- Seasonal patterns: peak heat May (30.8°C), coldest January (25.4°C)
- ACF analysis: lag-1 = 0.91 → strong temporal persistence
- Correlation matrix and scatter plots

### 03 — Preprocessing & Feature Engineering

**Preprocessing:**
- Duplicate check → 0 duplicates, continuous time index
- Missing values → 0 missing
- Outlier treatment (Winsorization p1–p99) on `solarradiation`, `sealevelpressure`, `dew`
- `temp` and `windspeed` outliers retained as genuine meteorological events

**Feature Engineering:**

| Group | Features | SARIMAX | XGBoost |
|---|---|---|---|
| Time features | month, quarter, dayofyear, season, sin/cos Fourier | ❌ | ✅ |
| Exogenous lags | dew_lag1, sealevelpressure_lag1 | ✅ | ✅ |
| Target lags | temp_lag1, temp_lag7 | ❌ | ✅ |
| Rolling mean | temp_rollmean7 | ❌ | ✅ |

> Scaling (StandardScaler) applied **after** train/test split, fitted on train set only.

### 04 — Modeling

**Train/Test Split** — Chronological, no shuffling
- Train: 2016-01-08 → 2019-12-31 (n = 1,454 | 82%)
- Test : 2020-01-01 → 2020-11-15 (n = 320  | 18%)

**Stationarity:** ADF test confirms stationarity in levels (ADF = −4.617, p < 0.001). d = 1 selected.

**Granger Causality Testing** — all 7 candidate variables tested:

| Variable | Min p-value | Decision |
|---|---|---|
| winddir | < 0.001 *** | Granger-causes temp ✓ |
| humidity | 0.0063 ** | Granger-causes temp ✓ |
| sealevelpressure | 0.0071 ** | Granger-causes temp ✓ |
| dew | 0.0315 * | Granger-causes temp ✓ |
| solarradiation | 0.5802 | No effect ✗ |
| windspeed | 0.2256 | No effect ✗ |
| preciptype | 0.0720 | No effect ✗ |

**SARIMA** — Grid search over p,q ∈ {0,1,2} and P,Q ∈ {0,1} (36 combinations)
→ Best: SARIMA(2,1,2)(0,1,1)₁₂ | AIC = 3470.46

**SARIMAX** — Same order + 4 Granger-significant exogenous variables
→ AIC = 789.87 (substantially lower than SARIMA)

**XGBoost** — 20 engineered features, early stopping at 30 rounds
→ Converges at iteration 298

**Forecast method:** SARIMA and SARIMAX use **rolling 1-step-ahead forecast**
(model re-fitted at each test step using actual past values) to avoid flat
long-horizon predictions.

---

## Results

| Metric | SARIMA | SARIMAX | XGBoost |
|---|---|---|---|
| Type | Statistical | Statistical | Machine Learning |
| Exogenous vars | None | 4 (Granger) | 20 features |
| AIC | 3470.46 | 789.87 | N/A |
| Train RMSE (°C) | 1.2212 | 0.9819 | 0.3533 |
| **Test MAE (°C)** | 0.6100 | **0.2310** | 0.3532 |
| **Test RMSE (°C)** | 0.8131 | **0.3129** | 0.5012 |
| **Test MAPE (%)** | 2.2155 | **0.8316** | 1.2899 |
| **Test R²** | 0.8362 | **0.9758** | 0.9378 |
| RMSE ratio | 0.67 | 0.32 | 1.42 |
| **Rank** | 🥉 3rd | 🥇 **1st** | 🥈 2nd |

> RMSE ratio = Test RMSE / Train RMSE

**Key finding:** SARIMAX with Granger-selected variables achieves 61.5% RMSE
reduction over SARIMA and outperforms XGBoost despite using only 4 features
vs 20 — demonstrating the value of causality-based variable selection.

---

## Setup & Usage

### 1. Clone the repository
```bash
git clone https://github.com/<your-username>/temperature-forecasting.git
cd temperature-forecasting
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Run notebooks in order
```bash
jupyter notebook
```

| Step | Notebook | Output |
|---|---|---|
| 1 | `01_data_understanding.ipynb` | Dataset overview |
| 2 | `02_eda.ipynb` | EDA figures (fig1–fig7) |
| 3 | `03_preprocess_fe.ipynb` | `rainfall_fe.csv` |
| 4 | `04_modeling.ipynb` | Model results & comparison |

---

## Dependencies

```
pandas>=1.5.0
numpy>=1.23.0
matplotlib>=3.6.0
seaborn>=0.12.0
scipy>=1.9.0
statsmodels>=0.14.0
xgboost>=1.7.0
scikit-learn>=1.1.0
```

Install:
```bash
pip install -r requirements.txt
```

---

## Notes

- **Scaling:** StandardScaler fitted on train set only, applied to both sets
- **Rolling forecast:** SARIMA/SARIMAX re-fitted at each step — avoids flat forecast problem
- **Windspeed outliers:** 33 monsoon-season values retained (Jun–Sep)
- **Temp outliers:** 23 cold-season readings retained (Jan–Feb)
- **Convergence warnings:** SARIMAX optimization produced warnings at a subset
  of rolling steps; results remain valid

---

## Report

A full academic report (LaTeX) is available in `report_full.tex`, compiled
following the econometric research writing guidelines of the National
Economics University (NEU).

---

*National Economics University (NEU) — Faculty of Mathematical Economics — May 2026*
