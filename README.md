# Residual Load Forecast for Luxembourg's power grid CREOS

The machine learning pipeline is engineered to predict the **residual load** (total electric power demand minus renewable power from wind and solar) for the Luxembourg TSO (**CREOS**). 

Accurate residual load forecasts allow grid operators to schedule dispatchable non-renewable power backups accordingly, resulting in minimized system imbalances. It also allows to anticipate and handle redispatches. Carbon emissions are reduced both in the short term by maximizing renewable energy output and support energy planning. This modeling pipeline benchmarks gradient-boosted ensemble models against the official prognosis published by **SMARD** (Bundesnetzagentur).

---

## 🏗️ Core Architecture & Pipeline Flow

The project is structured into four functional phases, displayed in the pipeline schematic:
<img width="1941" height="544" alt="GridForecastPipelineSchematic" src="https://github.com/user-attachments/assets/d8587d04-c800-47be-b383-562bfccca970" />

### 1. Data Staging & Chronological Alignment
* **Target Feature:** Collects hourly Residual load data
* **Spatial Feature Extraction:** Collects regional weather data from 8 strategic weather stations.
* **Cyclicity-Preserving Imputation:** High completeness of the dataset leads to missing weather entries getting reconstructed using **Cubic Spline Time Interpolation** to preserve time series properties and cyclic behavior.

### 2. Statistics & Physics-Driven Feature Engineering
To maximize the predictive capacity of tree-based architectures, raw variables are transformed into complex indicators:
* **Regional Mean and standard deviation:** Combining highly correlated features into condensed signals without losing predictive capabilities.
* **The Wind-Power Curve Factor ($v^3$):** Captures kinetic potential by cubing wind speeds and enforcing a standard $12\text{ m/s}$ mechanical turbine ceiling.
* **Thermal Heating Thresholds:** Calculates a metric to isolate heat pump usage patterns during the cold season using a hard threshold.
* **Autoregressive Features:** A 24-hour and 7-day lag residual load predicting column to account for human behavior cyclicity.
* **Datetime Augmentation:** Using anthropogenic patterns using calendaric inputs to better account for changing energy demands. 

### 3. Tree Ensembling & Tuning
Utilizing the flexibility of ensemble models combined with the efficiency of gradient boosting algorithms and RandomizedSearch strategy, this pipeline manages to catch a lot of complexities that factor into energy predictions with quick training:
* **Algorithms Evaluated:** `XGBoost`, `LightGBM`, and `HistGradientBoosting`.
* **Hyperparameter Optimization:** Utilizes a `RandomizedSearchCV` exploration loop evaluating different scalers and tree building parameters. 

### 4. Benchmark Validation
* **Validation Window:** Models are trained on historical authentic data stretching back to **January 2017** and evaluated on an out-of-time validation slice running from **June 2025 to May 2026**. 
* **Metrics Tracked:** Mean Absolute Error (MAE), Median Absolute Error (MedAE), Root Mean Squared Error (RMSE), Mean Absolute Percentage Error (MAPE) and Coefficient of Determination ($R^2$).
* **Baseline Target:** Directly challenges the accuracy of the official **SMARD** generation prognosis to prove real-world operational viability.

---

## 📊 Performance Summary & Key Results
The tuned models outperform the official prediction in squared metrics like RMSE and r² by a lot, while being only slightly behind in linear metrics (MAE+ MAPE). The Median though is a strong point for SMARD. 50% of the time their prediction is at least 25% closer to the actual residual load. 

| Model Configuration | MAE (MWh) | MedAE (MWh) | RMSE (MWh) | MAPE (MWh) | $R^2$ Score | 
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Official SMARD Prognosis** | **46.11** | **27.25** | 71.1 | **13.29** | 0.660 | 
| **LightGBM Regressor (Tuned)** | 47.77 | 36.86 | **63.71** | 14.12 | **0.727** |
| **XGBoost Regressor (Tuned)** | 48.23 | 37.56 | 63.86 | 14.05 | 0.726 |

This one-week snapshot highlights above summary regarding outlier and prediction consistency of the official prediction, the model prediction and ground truth. 
<img width="1589" height="490" alt="output_deepdive" src="https://github.com/user-attachments/assets/a8196bc5-ff12-4a12-9587-c20338c66b4d" />

## 🌐 Data Sources & Attribution
Weather data from: 'agrimeteo.lu' : https://www.agrimeteo.lu/Agrarmeteorologie/Wetterdaten/Download-Grafik (CC0 1.0) <br>
CREOS grid data from 'Bundesnetzagentur | SMARD.de' : https://www.smard.de/home/downloadcenter/download-marktdaten/ (CC BY 4.0)
