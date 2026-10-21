# Istanbul Water Supply Forecast

> **Disclaimer:** This is an independent data science and portfolio project. The time-series models, forecasts, and analyses presented herein were developed independently and do not constitute an official publication, forecast, or endorsement by the Istanbul Metropolitan Municipality (İBB) or İSKİ.

Forecasting the monthly volume of clean water supplied to Istanbul by İSKİ, using classical time series models (Holt-Winters and SARIMA) in Python.

---

## Overview

İSKİ's water supply figures have a strong upward trend and a clear summer peak. Forecasting them shows where demand is heading and gives a baseline for planning capacity. The dataset stops in January 2024, so this project forecasts the 24 months after it (February 2024 to January 2026).

* **Full Report:** 📄 [Full Project Report (PDF)](Istanbul%20Water%20Supply%20Forecast%20Report.pdf).
* **Code:** 📓 [Jupyter Notebook](iski-timeseries.ipynb).

---

## Data

* **Dataset:** Monthly water volumes supplied to regions in Istanbul (`ibb_su_veri.csv`), January 2009 to January 2024, 181 monthly observations after cleaning.
* **Source:** [Istanbul Metropolitan Municipality (İBB) Open Data Portal](https://data.ibb.gov.tr/dataset/istanbul-bolgelere-verilen-temiz-su-miktarlari)
* **License:** Distributed under the [İBB Open Data License](https://data.ibb.gov.tr/license) (adhering to Open Data Commons Attribution License guidelines).
* **Format:** Wide table with one row per month and one column per year, univariate (only the water volume, in m³).

---

## Method

* **Diagnosis:** Found five problems: wide format, a suspiciously low December 2019 value (51,278), mixed units (2009-2019 recorded in thousands, 2020 onward in full m³), empty months after January 2024, and year columns named like `2009.0`.
* **Cleaning:** Melted the table to long format, converted month names to numbers, built a monthly `DatetimeIndex`, multiplied 2009-2019 values by 1000 to match the later units, and dropped the empty future months.
* **Imputation:** December 2019 was replaced using the median ratio of December to the average of its neighbouring months (ratio ≈ 1.014 across the other years), giving 87,057,032. The imputed point sits on the series without breaking the pattern.
* **Exploration:** 12-month moving average and standard deviation, yearly and monthly boxplots. July and August are the peak months, and growth slows after 2020.
* **Decomposition:** STL on raw and log-transformed data. Seasonal peaks grow in the raw data (13.4M to 22.3M) but stay stable in the log data (0.229 to 0.243), so the seasonality is multiplicative.
* **Stationarity:** ADF and KPSS tests on four transformations. Only first differencing combined with seasonal differencing (d=1, D=1) passes both tests. ACF/PACF plots confirm yearly seasonality at lags 12, 24, and 36.
* **Split:** Trained on January 2009 to December 2021 (156 months), tested on January 2022 to January 2024 (25 months).
* **Models:** Seasonal Naive baseline, Holt-Winters (additive damped trend, multiplicative seasonality), and SARIMA on log data. For SARIMA, a grid search over 36 `(p,1,q)(P,1,Q,12)` combinations was ranked by AIC and then BIC.
* **Validation:** Rolling-origin test with six cutoffs (December 2017 to December 2022) and a 12-month horizon.
* **Final forecast:** Both models refit on all data and forecast 24 months, with a 95% interval for SARIMA.

---

## Results

### Hold-out test set (January 2022 to January 2024)

| Model | MAE | RMSE | MAPE |
| :--- | :--- | :--- | :--- |
| **Seasonal Naive** | 3,436,327 | 3,984,962 | 3.73% |
| **Holt-Winters** | 1,277,775 | 1,644,420 | 1.36% |
| **SARIMA (0,1,1)(0,1,1,12)** | 1,357,449 | 1,968,996 | 1.47% |

### Rolling-origin test (mean MAE over six cutoffs, 12-month horizon)

| Model | MAE |
| :--- | :--- |
| **Seasonal Naive** | 2,304,804 |
| **Holt-Winters** | 1,583,554 |
| **SARIMA** | 1,780,359 |

Both models beat the naive baseline, except in the 2020 window (a possible COVID effect). Holt-Winters is best in four of six windows, but the gap to SARIMA is small and driven mostly by the 2017 window, so the final forecast shows both.

---

## Limitations

* SARIMA tends to underestimate the summer peaks. The largest residuals are in July and August.
* The Holt-Winters damping parameter (0.993) is close to 1, so the trend is barely damped and long-horizon forecasts may keep growing too fast, even though the series is flattening after 2020.
* The difference between the two models is small, so neither is clearly better.
* Actual values for 2024 to 2026 are not in the dataset, so the final forecast could not be checked against real data.
* The December 2019 value was imputed from a ratio, not from a verified source.

---

## Tools

* 🐍 Python 3
* 📓 Jupyter Notebook
* 🐼 pandas
* 🔢 numpy
* 📊 matplotlib / seaborn
* 📈 statsmodels
* 🤖 scikit-learn (error metrics)

---

## How to Run

```bash
pip install pandas numpy matplotlib seaborn statsmodels scikit-learn
jupyter notebook iski-timeseries.ipynb
```
Place ibb_su_veri.csv in the same folder as the notebook.

## Author

Fikret Bartu Hıraekşi



