# 🌡️ Time-Series-Final
**Time Series Analysis using SARIMA, SARIMAX & XGBoost**

---

## 📌 Overview

This project analyzes and forecasts daily temperature using meteorological data collected from 2016 to 2020. Three models are built and compared: a univariate statistical model (SARIMA), a multivariate statistical model (SARIMAX), and a machine learning model (XGBoost). The study follows standard econometric research writing guidelines and is structured as an academic report.

---

## 📂 Repository Structure

```
📦 temperature-forecasting
 ├── 📓 01_data_understanding.ipynb     # Dataset overview, variable types, basic stats
 ├── 📓 02_eda.ipynb                    # Exploratory Data Analysis (7 figures)
 ├── 📓 03_preprocess_fe.ipynb          # Preprocessing & Feature Engineering
 ├── 📓 04_modeling.ipynb               # SARIMA, SARIMAX & XGBoost modeling
 ├── 📄 rainfall.csv                    # Raw dataset
 ├── 📄 rainfall_fe.csv                 # Processed dataset (output of notebook 03)
 ├── 📄 requirements.txt                # Python dependencies
 └── 📄 README.md
```

---

## 📊 Dataset

| Attribute | Detail |
|---|---|
| **Source** | Daily meteorological observations |
| **Period** | 2016-01-01 → 2020-11-15 |
| **Observations** | 1,781 days |
| **Target variable** | `temp` — Daily temperature (°C) |
| **Features** | `dew`, `humidity`, `sealevelpressure`, `winddir`, `solarradiation`, `windspeed`, `preciptype` |

---

## 🔬 Methodology

### 1. Data Understanding
- Variable types, distributions, and basic descriptive statistics
- Time range and frequency verification

### 2. Exploratory Data Analysis
- Distribution analysis (histogram, boxplot, Q-Q plot)
- Time series decomposition with rolling mean and trend
- Seasonality patterns by month and quarter
- Autocorrelation Function (ACF) — lag-1 = 0.91
- Correlation matrix and scatter plots vs target

### 3. Preprocessing
- **Duplicate check** — 0 duplicates, continuous time index
- **Missing values** — 0 missing values found
- **Outlier treatment** — Winsorization at p1–p99 for `solarradiation`, `sealevelpressure`, `dew`; `temp` and `windspeed` outliers retained as genuine meteorological events

### 4. Feature Engineering

| Feature Group | Features | Used by |
|---|---|---|
| Time features | month, quarter, dayofyear, season, sin/cos Fourier | XGBoost |
| Exogenous lags | dew_lag1, sealevelpressure_lag1 | SARIMAX, XGBoost |
| Temp lags | temp_lag1, temp_lag7 | XGBoost |
| Rolling mean | temp_rollmean7 | XGBoost |

> Scaling (StandardScaler) applied after train/test split, fitted on train set only to prevent data leakage.

### 5. Modeling

**Train/Test Split** — Chronological (no shuffling)
- Train: 2016-01-08 → 2019-12-31 (~82%)
- Test : 2020-01-01 → 2020-11-15 (~18%)

**SARIMA** `(p,d,q)(P,D,Q,12)`
- Parameters selected via grid search (AIC minimization)
- ADF test: original series stationary at α=0.05
- Forecast method: 1-step ahead rolling forecast

**SARIMAX** `(p,d,q)(P,D,Q,12) + exog`
- Exogenous variables: `dew`, `sealevelpressure` (scaled)
- Same order as SARIMA from grid search
- Forecast method: 1-step ahead rolling forecast

**XGBoost**
- 20 engineered features
- Regularization: `max_depth=3`, `reg_alpha=0.1`, `reg_lambda=1.5`
- Early stopping on validation RMSE

---

## 📈 Results

| Metric | SARIMA | SARIMAX | XGBoost |
|---|---|---|---|
| **Type** | Statistical | Statistical | Machine Learning |
| **Test MAE (°C)** | 0.6100 | 0.6116 | 0.3530 |
| **Test RMSE (°C)** | 0.8131 | 0.8166 | 0.5010 |
| **Test MAPE (%)** | 2.2155 | 2.2222 | — |
| **Test R²** | 0.8362 | 0.8348 | 0.9380 |
| **RMSE ratio** | 0.67 | 0.67 | — |
| **Rank** | 🥈 2nd | 🥉 3rd | 🥇 1st |

**Key findings:**
- XGBoost achieves highest accuracy (R²=0.938) by leveraging lag and rolling features
- SARIMAX does not improve over SARIMA — exogenous variables add limited predictive value beyond the autoregressive structure
- SARIMA remains competitive (R²=0.836) with no feature engineering required

---

## ⚙️ Setup & Usage

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

| Order | Notebook | Output |
|---|---|---|
| 1 | `01_data_understanding.ipynb` | Dataset overview |
| 2 | `02_eda.ipynb` | EDA figures (fig1–fig7) |
| 3 | `03_preprocess_fe.ipynb` | `rainfall_fe.csv` |
| 4 | `04_modeling.ipynb` | Model results & comparison figures |

> ⚠️ Notebooks must be run **in order** — each notebook depends on outputs from the previous one.

---

## 📦 Dependencies

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

---

## 📝 Notes

- **Scaling:** StandardScaler is fitted on the train set only and applied to both train and test to prevent data leakage
- **Rolling forecast:** SARIMA and SARIMAX use 1-step ahead rolling forecast — model re-fitted at each step using actual past values to avoid the flat forecast problem from long out-of-sample prediction
- **Windspeed outliers:** 33 observations above IQR upper fence retained as natural monsoon wind events (Jun–Sep)
- **Temp outliers:** Cold-season readings (Jan–Feb) retained as genuine meteorological events

---

## 📚 Reference

Adapted from *Writing Economics: A Guide for Harvard Economics Concentrators*

