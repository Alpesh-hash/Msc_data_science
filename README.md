# Short-term NO₂ Forecasting at London Bloomsbury (CLL2) using a Naïve Baseline, XGBoost and LSTM

## Project overview

This project investigates **one-hour-ahead forecasting of nitrogen dioxide (NO₂)** at the **London Bloomsbury monitoring station (CLL2)** using hourly air-quality observations from **UK-AIR/DEFRA** covering **2015–2025**.

The main aim of the project is to compare whether **machine-learning** and **deep-learning** methods provide meaningful improvement over a simple **naïve persistence baseline** for short-horizon site-level forecasting.

The three models compared are:

- **Naïve persistence baseline**
- **XGBoost**
- **LSTM (Long Short-Term Memory)**

The project was developed in **Python** using **Google Colab**.

---

## Research question

To what extent do machine-learning and deep-learning models improve one-hour-ahead NO₂ forecasting at London Bloomsbury (CLL2) compared with a simple persistence baseline, when trained and evaluated on a strict chronological split?

---

## Dataset

The dataset was obtained from the **UK Air Information Resource (UK-AIR)**, maintained on behalf of **DEFRA**.

### Monitoring site
- **London Bloomsbury (CLL2)**

### Variables used
- **NO₂**
- **NO**
- **NOx (as NO₂)**
- **O₃**

### Time span
- **2015–2025**

### Notes
The data were handled as hourly time series. Timestamps were standardised, reindexed to a complete hourly grid, and short gaps of up to three hours were interpolated conservatively. Longer gaps were left missing.

---

## Method summary

### 1. Preprocessing
- Standardised date/time fields, including handling of `"24:00"` timestamps
- Reindexed to a full hourly timeline
- Converted pollutant columns to numeric values
- Interpolated only short gaps (up to 3 hours)
- Left longer gaps missing
- Performed exploratory analysis of missingness, seasonality, and pollutant behaviour

### 2. Forecasting setup
- Forecasting target: **NO₂ at t+1**
- Strict chronological split:
  - **Training:** earlier years
  - **Validation:** 2022–2023
  - **Test:** 2024–2025

### 3. Models
- **Naïve baseline:** predicts next-hour NO₂ as the current NO₂ value
- **XGBoost:** trained on engineered tabular features
- **LSTM:** trained on contiguous multivariate sequence windows

### 4. Evaluation
Models were evaluated using:
- **MAE** (Mean Absolute Error)
- **RMSE** (Root Mean Squared Error)

Both metrics were reported in original NO₂ units.

---

## Feature engineering

### XGBoost
The XGBoost model used engineered predictors including:
- lagged pollutant values
- rolling summary statistics
- exponentially weighted features
- cyclical calendar features such as hour-of-day encodings

### LSTM
The LSTM model used:
- ordered multivariate sequence windows
- contiguous valid hourly segments only
- training-only scaling parameters applied to validation and test data

This segment-aware construction prevented false sequence continuity across long missing gaps.

---

## Final model settings

### XGBoost
The XGBoost model was tuned using validation MAE with early stopping. The search covered:
- learning rate
- max depth
- min child weight
- subsample
- colsample
- gamma
- regularisation

Best iteration was identified at approximately **271 boosting rounds**.

### LSTM
The final tuned LSTM configuration used:
- **48-hour input window**
- **96 hidden units**
- **dropout = 0.0**
- **learning rate = 0.001**
- **batch size = 64**
- **MAE loss**
- **stacked architecture**

Training stability was improved using:
- **EarlyStopping**
- **ReduceLROnPlateau**

---

## Final results

| Model | Val MAE | Val RMSE | Test MAE | Test RMSE |
|------|--------:|---------:|---------:|----------:|
| Naïve persistence | 4.0436 | 6.0033 | 3.6270 | 5.3670 |
| XGBoost | 3.7464 | 5.5293 | 3.4720 | 5.0286 |
| LSTM | 3.5373 | 5.3198 | 3.2418 | 4.8441 |

### Main finding
Both learned models outperformed the naïve persistence baseline, and the **LSTM achieved the best overall performance**, with **XGBoost as a strong second-best model**.

---

## Repository structure

Example structure:

```text
.
├── README.md
├── notebooks/
│   └── Msc_final_project_revised_v2_segmented_v3_kerastuner_fast.ipynb
├── data/
│   ├── CLL2_2015.csv
│   ├── CLL2_2016.csv
│   ├── ...
│   └── CLL2_2025.csv
├── figures/
│   ├── missingness_by_pollutant.png
│   ├── no2_hourly_pattern.png
│   ├── lstm_training_history.png
│   ├── lstm_validation_plot.png
│   └── lstm_test_plot.png
└── report/
    └── final_report.docx
