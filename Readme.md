# Weather Trend Forecasting

A data-science project that analyzes global weather patterns and forecasts future daily temperatures using the **Global Weather Repository** dataset.

The project covers data preprocessing, exploratory data analysis, anomaly detection, geographical and air-quality analysis, feature importance, multiple forecasting models, ensemble forecasting, and a final seven-day temperature forecast.

---

## Project Overview

The objective of this project is to analyze daily weather observations from cities around the world and develop a reliable temperature-forecasting workflow.

The analysis includes:

* Data cleaning and preprocessing
* Missing-value and outlier handling
* Temperature and precipitation analysis
* Correlation and seasonal analysis
* Anomaly detection using Isolation Forest
* Country, continent, latitude, and spatial comparisons
* Weather and air-quality relationship analysis
* Time-series feature engineering
* Multiple forecasting models
* Ensemble forecasting
* Feature-importance analysis
* Seven-day future temperature forecasting

The `last_updated` column was used as the primary time variable for time-series analysis.

---

## Dataset

**Dataset:** Global Weather Repository
**Source:** [Global_Weather_Dataset](https://www.kaggle.com/datasets/nelgiriyewithana/global-weather-repository)

The dataset contains daily weather information for cities around the world, including:

* Temperature
* Precipitation
* Humidity
* Atmospheric pressure
* Wind speed and direction
* Cloud cover
* Visibility
* Latitude and longitude
* Air-quality measurements
* Astronomical information

The main forecasting target used in this project was:

```text
temperature_celsius
```

The dataset file is not included in this repository. Download it from Kaggle and place it in the project data directory before running the notebook.

---

## Project Workflow

The project follows the workflow below:

```text
Data Loading
    ↓
Data Inspection and Validation
    ↓
Data Cleaning and Preprocessing
    ↓
Exploratory Data Analysis
    ↓
Anomaly Detection
    ↓
Climate and Geographical Analysis
    ↓
Air-Quality and Spatial Analysis
    ↓
Forecasting Data Preparation
    ↓
Baseline and Advanced Models
    ↓
Ensemble Forecasting
    ↓
Feature-Importance Analysis
    ↓
Final Seven-Day Forecast
```

---

## Data Preparation

The preprocessing workflow included:

* Converting `last_updated` to datetime format
* Sorting observations chronologically
* Checking missing values
* Checking duplicate records
* Validating numerical ranges
* Flagging temperature outliers using the IQR method
* Retaining plausible extreme-weather observations
* Creating calendar features
* Creating a continuous daily temperature series
* Applying feature scaling within the Ridge Regression pipeline

For the selected forecasting location, three missing calendar dates were filled using time-based interpolation.

---

## Exploratory Data Analysis

The exploratory analysis examined:

* Global temperature distributions
* Daily and monthly temperature trends
* Precipitation distributions and patterns
* Relationships among weather variables
* Seasonal temperature differences
* Country and continent comparisons
* Latitude and temperature relationships
* Spatial temperature distribution
* Weather and air-quality relationships

The analysis showed clear geographical and temporal differences in temperature, precipitation, and environmental conditions.

---

## Anomaly Detection

Isolation Forest was used to detect unusual combinations of:

* Temperature
* Humidity
* Precipitation
* Atmospheric pressure
* Wind speed
* Cloud cover
* Visibility

The detected anomalies were retained because they could represent genuine unusual weather conditions, temporary local events, or measurement irregularities.

---

## Forecasting Location

Forecasting was performed for:

```text
Bujumbura, Burundi
```

Bujumbura was selected because it contained the longest suitable daily temperature history in the dataset.

| Data Stage                  | Observations |
| --------------------------- | -----------: |
| Original recorded dates     |          794 |
| Missing calendar dates      |            3 |
| Continuous daily series     |          797 |
| Final modeling observations |          783 |

Historical period:

```text
May 16, 2024 – July 21, 2026
```

---

## Feature Engineering

The following forecasting features were created:

| Feature          | Description                                        |
| ---------------- | -------------------------------------------------- |
| `lag_1`          | Temperature one day earlier                        |
| `lag_7`          | Temperature seven days earlier                     |
| `lag_14`         | Temperature fourteen days earlier                  |
| `rolling_mean_7` | Mean temperature over the previous seven days      |
| `rolling_std_7`  | Temperature variation over the previous seven days |
| `month`          | Calendar month                                     |
| `day_of_year`    | Position within the year                           |
| `day_of_week`    | Day of the week                                    |

The first 14 observations were removed after feature engineering because `lag_14` was unavailable for those rows.

---

## Chronological Data Split

The data was split chronologically to prevent future observations from influencing model training.

| Dataset    | Observations | Date Range                         |
| ---------- | -----------: | ---------------------------------- |
| Training   |          548 | May 30, 2024 – November 28, 2025   |
| Validation |          117 | November 29, 2025 – March 25, 2026 |
| Test       |          118 | March 26, 2026 – July 21, 2026     |

The validation set was used for model comparison and ensemble-weight calculation. The test set was reserved for final evaluation.

---

## Forecasting Models

The following models were evaluated:

### Naive Persistence

Uses the previous day’s temperature as the next prediction.

### Ridge Regression

Uses standardized lag, rolling, and calendar features to model linear relationships.

### Random Forest

Uses multiple decision trees to capture nonlinear relationships among the engineered features.

### SARIMA

Models the ordered temperature series using autoregressive, moving-average, and weekly seasonal components.

Configuration:

```text
SARIMA(1,0,1)(1,0,1,7)
```

### Ensemble Models

Two ensemble approaches were tested:

* Simple Average Ensemble
* Validation-Weighted Ensemble

The weighted ensemble assigned larger weights to models with lower validation MAE.

---

## Evaluation Metrics

The models were evaluated using:

* Mean Absolute Error
* Root Mean Squared Error
* R-squared
* Symmetric Mean Absolute Percentage Error

Lower MAE, RMSE, and sMAPE values indicate better forecasting performance.

---

## Final Model Results

| Rank | Model                        |         MAE |        RMSE |        R² |      sMAPE |
| ---: | ---------------------------- | ----------: | ----------: | --------: | ---------: |
|    1 | **SARIMA**                   | **1.145°C** | **1.521°C** | **0.083** | **5.097%** |
|    2 | Simple Average Ensemble      |     1.158°C |     1.562°C |     0.034 |     5.150% |
|    3 | Validation-Weighted Ensemble |     1.160°C |     1.572°C |     0.021 |     5.158% |
|    4 | Ridge Regression             |     1.195°C |     1.597°C |    -0.010 |     5.307% |
|    5 | Random Forest                |     1.234°C |     1.657°C |    -0.088 |     5.478% |
|    6 | Naive Persistence            |     1.446°C |     1.960°C |    -0.521 |     6.453% |

SARIMA achieved the strongest final test performance.

Compared with the naive persistence baseline, SARIMA reduced:

* MAE by approximately **20.8%**
* RMSE by approximately **22.4%**
* sMAPE by approximately **21.0%**

Both ensemble approaches outperformed the naive baseline but did not outperform standalone SARIMA.

---

## Feature Importance

Feature importance was evaluated using:

* Correlation analysis
* Random Forest built-in importance
* Permutation importance

| Method                   | Most Important Feature |
| ------------------------ | ---------------------- |
| Correlation              | `rolling_mean_7`       |
| Random Forest importance | `rolling_mean_7`       |
| Permutation importance   | `lag_1`                |

The results showed that recent temperature history was more useful than calendar information.

The previous seven-day average and the previous day’s temperature were the strongest forecasting features.

---

## Seven-Day Forecast

After evaluation, SARIMA was retrained using all 797 available daily observations.

The model generated forecasts for:

```text
July 22, 2026 – July 28, 2026
```

| Date          | Predicted Temperature | Lower 95% Interval | Upper 95% Interval |
| ------------- | --------------------: | -----------------: | -----------------: |
| July 22, 2026 |               20.94°C |            16.38°C |            25.50°C |
| July 23, 2026 |               21.14°C |            16.52°C |            25.75°C |
| July 24, 2026 |               21.00°C |            16.33°C |            25.67°C |
| July 25, 2026 |               21.13°C |            16.40°C |            25.85°C |
| July 26, 2026 |               21.06°C |            16.28°C |            25.84°C |
| July 27, 2026 |               21.18°C |            16.35°C |            26.01°C |
| July 28, 2026 |               20.95°C |            16.06°C |            25.83°C |

Forecast summary:

* Minimum predicted temperature: **20.94°C**
* Maximum predicted temperature: **21.18°C**
* Average predicted temperature: **21.06°C**

The point forecasts remained stable, while the wider prediction intervals represented uncertainty in future daily temperatures.

---

## Repository Structure

```text
Weather-Forecasting/
│
├── weather_trend_forecasting.ipynb
├── Weather_Trend_Forecasting_Report.pdf
├── README.md
├── requirements.txt
├── .gitignore
```

The dataset is not stored in the repository and must be downloaded separately from Kaggle.

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/rajaraman04/Weather-Forecasting
cd Weather-Forecasting
```

### 2. Create a virtual environment

Windows:

```bash
python -m venv .venv
.venv\Scripts\Activate.ps1
```

macOS or Linux:

```bash
python3 -m venv .venv
source .venv/bin/Activate.ps1
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Download the dataset

Download the Global Weather Repository CSV file from Kaggle.

Place the CSV file in the location expected by the notebook, or update the dataset path in the data-loading section.

### 5. Run the notebook

```bash
jupyter notebook weather_trend_forecasting.ipynb
```

The notebook can also be opened and executed using the Jupyter extension in Visual Studio Code.

---

## Main Libraries

* NumPy
* Pandas
* Matplotlib
* Seaborn
* Plotly
* Scikit-learn
* Statsmodels
* Country Converter
* Jupyter Notebook

Recommended Python version:

```text
Python 3.12
```

---

## Report

The detailed analysis, interpretations, methodology, findings, and limitations are available in the project report:

```text
Weather_Trend_Forecasting_Report.pdf
```


## PM Accelerator Mission

By making industry-leading tools and education available to individuals from all backgrounds, we level the playing field for future PM leaders. This is the PM Accelerator motto, as we grant aspiring and experienced PMs what they need most – Access. We introduce you to industry leaders, surround you with the right PM ecosystem, and discover the new world of AI product management skills.

---

## Author

**Rajaraman Rajagopalan**

* Portfolio: https://rajaraman.dev
* LinkedIn: https://www.linkedin.com/in/rajaraman04

---
