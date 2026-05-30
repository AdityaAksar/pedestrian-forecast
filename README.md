# Melbourne Pedestrian Forecasting Model Development

This repository contains the exploratory notebook used for EDA, model development, and comparison for the Melbourne Pedestrian Analytics project. The final XGBoost model from this notebook was productionized in the [pedestrian-pipeline](https://github.com/AdityaAksar/pedestrian-pipeline) repository.

---

## Notebook

`pedestrian_forecasting.ipynb`

### Contents

| Section | Description |
|---|---|
| 1. Setup & Authentication | Install dependencies, authenticate to Google BigQuery via Colab |
| 2. Data Loading | Load `mart_pedestrian_hourly` from BigQuery (~1.6M rows) |
| 3. EDA | Daily trends, hourly patterns, day-of-week patterns, top 10 busiest sensors |
| 4. Data Preparation | Filter to sensor SouthB_T (Riverside, location_id 212) as representative sensor |
| 5. Model 1 — Prophet | Baseline time series model |
| 6. Model 2 — XGBoost | Tree-based model with lag features and weather data |
| 7. Evaluation | Side-by-side comparison of both models on test set |
| 8. Save Predictions | Export XGBoost predictions to BigQuery `mart.predictions` |

---

## Model Comparison

Both models were evaluated on the same 20% test set (most recent data).

| Model | MAE | RMSE | MAPE |
|---|---|---|---|
| Prophet | 445.8 | 593.8 | 186.3% |
| **XGBoost** | **92.3** | **145.9** | **26.5%** |

XGBoost outperforms Prophet by **4.8x on MAE**. Prophet struggles with the high variability and irregular spikes in pedestrian data, while XGBoost captures patterns effectively through engineered features.

---

## Feature Engineering (XGBoost)

| Feature | Description |
|---|---|
| `hour` | Hour of day (0-23) |
| `day_of_week` | Day of week (0=Monday, 6=Sunday) |
| `month` | Month of year |
| `quarter` | Quarter of year |
| `is_weekend` | Binary flag for Saturday and Sunday |
| `is_public_holiday` | Binary flag for Victorian public holidays |
| `temperature_c` | Hourly temperature in Celsius |
| `precipitation_mm` | Hourly precipitation in mm |
| `windspeed_ms` | Hourly wind speed in m/s |
| `humidity_pct` | Hourly relative humidity percentage |
| `lag_1h` | Pedestrian count 1 hour prior |
| `lag_24h` | Pedestrian count 24 hours prior |
| `lag_168h` | Pedestrian count 168 hours (1 week) prior |
| `rolling_mean_24h` | 24-hour rolling average |
| `rolling_mean_168h` | 168-hour rolling average |

---

## Data Source

Data is loaded from BigQuery mart layer produced by [pedestrian-dbt](https://github.com/AdityaAksar/pedestrian-dbt):

- Table: `melbourne-pedestrian-pipeline.staging.mart_pedestrian_hourly`
- Rows: ~1.6M
- Coverage: 100+ sensors, multiple years of hourly data

---

## Requirements

The notebook runs on Google Colab with the following dependencies:

```
google-cloud-bigquery
xgboost
scikit-learn
pandas
numpy
prophet
matplotlib
seaborn
```

---

## Relation to Production Pipeline

This notebook served as the development environment for the forecasting model. The final model was adapted into a production script (`run_forecast.py`) in the `pedestrian-pipeline` repository with the following additions:

- Loops through all 101 sensors automatically instead of a single sensor
- Uses Open-Meteo API for real weather forecast data instead of historical medians
- Writes results to BigQuery with WRITE_TRUNCATE for daily refresh
- Runs automatically via GitHub Actions every day after ingestion completes