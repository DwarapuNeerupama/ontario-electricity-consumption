# Ontario Electricity Consumption Prediction

Predicting monthly electricity consumption for commercial buildings in Ontario using the
ENERGY STAR Portfolio Manager dataset, with time series forecasting of provincial demand.

---

## Project Overview

This project has two parts:

**Part 1 — Per-building prediction**  
Build and compare three regression models to predict how much electricity a commercial
property will consume in a given month, based on building characteristics like floor area,
property type, occupancy, and seasonal factors.

**Part 2 — Time series forecasting**  
Aggregate all buildings by month and forecast total Ontario commercial electricity
consumption 12 months ahead using ARIMA and Holt-Winters.

**Dataset:** Ontario ENERGY STAR Portfolio Manager export (4 CSV files)  
**Target variable:** Monthly electricity usage in kWh (Electric – Grid meters only)

---

## Files in this repo

```
ontario-electricity-consumption/
├── electricity_consumption_prediction.ipynb   ← main notebook
├── Properties.csv
├── Meters.csv
├── Meter_Entries.csv
├── Uses.csv
└── README.md
```

---

## Part 1 — Regression Models

### Methods

**Data preparation**
- Replaced 'Not Available' placeholders with NaN across all four source files
- Filtered meter entries to Electric – Grid meters in kWh only
- Merged all four files on Portfolio Manager ID
- Outlier removal using log-scale boxplot inspection (removed readings below log=2 and above log=14)
- Median imputation for remaining missing numeric values

**Feature engineering**
- Extracted month and year from billing dates
- Created season feature (1=winter, 2=spring, 3=summer, 4=fall)
- Calculated days in billing period
- Label encoded property type
- Pivoted use-type floor areas (office, retail, etc.) as separate features

**Models compared**
- Ridge Regression (regularized linear baseline, with StandardScaler)
- Random Forest Regressor (150 trees, max depth 12, min samples leaf 3)
- Gradient Boosting Regressor (150 estimators, learning rate 0.1, max depth 5)

All models trained on log-transformed target (log1p kWh) to handle right skew.

### Results

| Model | R² | RMSE (kWh) | MAE (kWh) |
|---|---|---|---|
| Ridge Regression | 0.2266 | 1,004,471 | 59,409 |
| Random Forest | 0.5193 | 12,823 | 5,493 |
| **Gradient Boosting** | **0.5452** | **13,766** | **6,232** |

**Gradient Boosting performed best** with R²=0.55. Ridge underperformed significantly —
this is expected because electricity consumption has a non-linear relationship with building
size, property type, and season that a linear model can't fully capture.

---

## Part 2 — Time Series Forecasting

### Methods
- Aggregated all buildings into one total monthly consumption time series (11 months: Jan–Nov 2022)
- Train/test split by time (last 3 months as test — never split time series randomly)
- **ARIMA(1,1,1)** — captures trend and autocorrelation
- **Holt-Winters** — trend-only smoothing (seasonal component requires 24+ months of data)

### Results

| Model | RMSE | MAE | MAPE |
|---|---|---|---|
| **ARIMA(1,1,1)** | **1,125,052** | **912,829** | **5.37%** |
| Holt-Winters (trend only) | 1,889,050 | 1,390,488 | 7.98% |

**ARIMA outperformed Holt-Winters** on this dataset. The 12-month forward forecast
projects a gradual decline from ~17.85M kWh (Dec 2022) to ~16.28M kWh (Nov 2023).

---

## Key Findings

- **Gross floor area** is the strongest single predictor of electricity consumption —
  larger buildings consume more power, almost linearly
- **Property type** significantly influences consumption — industrial and large office
  buildings consume far more than residential or small retail
- **Seasonal patterns** are visible — Ontario winters and summers drive consumption peaks
  due to heating and cooling loads
- **Non-linear models** (Random Forest, Gradient Boosting) substantially outperform Ridge,
  confirming that consumption depends on complex interactions between features, not just
  simple linear relationships
- **ARIMA** is better suited for this short time series than Holt-Winters, which needs
  at least 24 months to fit a seasonal component

---

## How to Run

1. Clone this repo
2. Place all 4 CSV files in the same folder as the notebook
3. Install dependencies:
```
pip install pandas numpy scikit-learn matplotlib seaborn statsmodels
```
4. Open `electricity_consumption_prediction.ipynb` in Jupyter or VS Code
5. Run all cells top to bottom

---

## Tools Used

Python · Pandas · NumPy · scikit-learn · statsmodels · Matplotlib · Seaborn · Jupyter Notebook

---

## About

Personal data analytics project — I'm a Master of Data Analytics student at the
University of Niagara Falls Canada, building a portfolio targeting Fall 2026 co-op
and internship opportunities in data analytics across Ontario.

