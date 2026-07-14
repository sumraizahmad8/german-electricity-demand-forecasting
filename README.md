# Forecasting German Electricity Demand
 
A comparative case study forecasting German national electricity load using benchmark, statistical, machine learning, and deep learning models.
 
This repository contains the full analysis pipeline: data acquisition, exploratory analysis, stationarity testing, and six forecasting models (Mean, Naive, Seasonal Naive, Drift, SARIMA, SARIMAX, Random Forest, and LSTM), evaluated on an identical two-year forecast horizon.
 
## Project Structure
 
```
.
├── README.md
├── notebooks/
│   └── German_Electricity_Demand_Forecasting.ipynb   # Main analysis notebook (all parts)
├── data/
│   └── (raw data is downloaded at runtime — see below, not stored in repo)
├── figures/
│   └── (all generated plots, saved automatically when the notebook runs)
├── results/
│   └── metrics_table.csv                              # Final model comparison table
└── report/
    └── German_Electricity_Demand_Forecasting_Report.docx
```
 
## Data Sources
 
| Source | Description | Access |
|---|---|---|
| [Open Power System Data (OPSD)](https://data.open-power-system-data.org/time_series/) | Hourly German electricity load, 2015–2020 | Public CSV, downloaded directly in the notebook |
| [Open-Meteo Archive API](https://archive-api.open-meteo.com/v1/archive) | Historical Berlin temperature (proxy for Germany) | Public API, queried directly in the notebook |
 
No data files are committed to this repository — the notebook downloads both datasets directly when run, so the pipeline is fully reproducible from a clean clone.
 
## How to Run
 
1. Clone the repository:
```bash
   git clone <repo-url>
   cd <repo-name>
```
 
2. Install dependencies:
```bash
   pip install -r requirements.txt
```
 
3. Open and run the notebook top to bottom:
```bash
   jupyter notebook notebooks/German_Electricity_Demand_Forecasting.ipynb
```
   Or open it directly in [Google Colab](https://colab.research.google.com/).
 
   **Note:** the SARIMA grid search (Part 3) and LSTM hyperparameter comparison (Part 6) are computationally heavy and may take several hours on CPU. See the notebook markdown cells for expected runtimes at each step.
 
## Pipeline Overview
 
| Part | Description |
|---|---|
| 1 | Data acquisition, cleaning, resampling (hourly/daily/weekly), EDA, seasonal decomposition, stationarity testing (ADF/ACF/PACF) |
| 2 | Benchmark models: Mean, Naive, Seasonal Naive, Drift |
| 3 | SARIMA — grid search over (p,d,q)(P,D,Q) via AIC, residual diagnostics, forecast |
| 4 | SARIMAX — Berlin temperature added as an exogenous regressor |
| 5 | Random Forest — feature-engineered weekly model (lags, rolling stats, calendar, temperature) |
| 6 | LSTM — hourly-resolution sequence model with hyperparameter comparison |
| 7 | Critical analysis answering the assignment's six evaluation questions |
| 8 | Final report (Word document) |
 
## Key Results
 
All models were evaluated on the same 104-week (2-year) held-out test horizon. Full results are in [`results/metrics_table.csv`](results/metrics_table.csv).
 
| Model | RMSE | MAE | MAPE (%) |
|---|---|---|---|
| Seasonal Naive | 454,051 | 342,864 | 3.86 |
| Random Forest | 456,014 | 336,857 | 3.86 |
| LSTM (hourly → weekly) | 522,268 | 420,910 | 4.41 |
| Drift | 727,257 | 620,585 | 6.82 |
| Mean | 739,222 | 629,534 | 6.91 |
| Naive | 749,961 | 640,567 | 7.07 |
| SARIMA(4,1,6)(1,1,1,52) | 770,574 | 636,356 | 7.18 |
| SARIMAX (+Temperature, conditional) | 883,644 | 749,962 | 8.43 |
 
**Headline finding:** Seasonal Naive is a very strong benchmark for this series given its dominant annual seasonality. Only Random Forest matches it, largely because its feature importances show the 52-week load lag dominates its predictions (78% importance) — it has effectively re-derived the seasonal-naive heuristic. SARIMAX (with temperature) underperforms plain SARIMA, a result discussed in detail in the report (Section 5.4) — the covariate reinforces an incorrect seasonal-normality assumption during the COVID-19 period, when the actual demand shock was unrelated to temperature.
 
Full methodology, justification of modelling choices, and critical discussion are in the [report](report/German_Electricity_Demand_Forecasting_Report.docx).
 
## Known Limitations
 
- SARIMA's seasonal order search was constrained to P, Q ∈ {0,1} for computational tractability (see report Section 6 for full discussion).
- The LSTM uses only univariate load history; it does not include temperature or calendar features, unlike the Random Forest model.
- Neither SARIMAX nor the LSTM explicitly account for holiday effects.
- Temperature is used at its *observed* (not forecast) value in the test period — SARIMAX and Random Forest results involving temperature are therefore *conditional/explanatory* forecasts, not true operational forecasts.
See the report's Limitations and Future Work section for the complete discussion.
 
## Requirements
 
```
pandas
numpy
matplotlib
requests
statsmodels
scikit-learn
tensorflow
scipy
```
 
## Author
Sumraiz Ahmad
