# TCYFF — 25-Minute Presentation Notes
## Points with a short explanation for each part

Use these as speaking prompts. Read the main point, then explain it naturally in your own words.

---

# 0:00–1:30 — Introduction, problem, and solution

## Project name

- **Tunisia Cereal Yield Forecasting Framework, or TCYFF**
  - An end-to-end machine-learning decision-support system for predicting cereal yield in Tunisian governorates.

## Prediction target

- **Target:** `Target_Total_Yield_QxHa`
  - It combines durum wheat, soft wheat, and barley.

## Unit

- **Unit:** quintals per hectare, written as `qx/ha`
  - One quintal is 100 kilograms.
  - The unit shows how much cereal is produced from one hectare.

## The problem

- **Yield changes between governorates and years**
  - Rainfall, temperature, vegetation, soil, regional conditions, and past agricultural performance can all affect the result.

- **A national average can hide local problems**
  - One governorate may have poor yield even when the national result looks acceptable.

## Why the problem matters

- **Food planning becomes more difficult**
  - Decision-makers need an idea of which years or regions may produce less cereal.

- **Resource allocation becomes more difficult**
  - Agricultural support and monitoring may need to focus more on certain governorates.

## The solution

- **Build a governorate-level forecasting system**
  - The system combines agricultural and environmental data, predicts yield, and presents the results through a dashboard and API.

## Beneficiaries

- **Agricultural analysts** — study yield patterns and compare regions.
- **Government planners** — identify areas that may need more attention.
- **Researchers and universities** — study the data and machine-learning process.
- **Agricultural organisations** — support regional monitoring and planning.
- **Farmers indirectly** — may benefit from better decisions, although the model is not at farm level.

## Important limitation

- **Decision-support, not an official forecasting system**
  - The output should be combined with official statistics, field observations, and expert judgement.

---

# Technology stack

- **Python**
  - Main programming language for data preparation, modelling, testing, the dashboard, and the API.

- **pandas**
  - Reads, cleans, sorts, groups, and analyses the governorate-year dataset.

- **NumPy**
  - Supports numerical calculations and missing numeric values.

- **scikit-learn**
  - Provides preprocessing, pipelines, models, validation, and evaluation metrics.

- **ExtraTrees Regressor**
  - The final selected model used to predict cereal yield.

- **XGBoost and LightGBM**
  - Candidate models used during comparison and experimentation.

- **Joblib**
  - Saves and reloads the complete preprocessing and model pipeline.

- **Jupyter Notebook / Google Colab**
  - Used for dataset building, model comparison, analysis, and report figures.

- **Matplotlib**
  - Creates static charts and report figures.

- **Plotly**
  - Creates interactive charts in the dashboard.

- **Streamlit**
  - Provides the interactive dashboard for human users.

- **FastAPI**
  - Provides the REST API for software access to model information and predictions.

- **Uvicorn**
  - Runs the FastAPI application locally.

- **Pydantic**
  - Checks the structure of API requests.

- **JSON**
  - Stores metadata and is used for API requests and responses.

- **pytest**
  - Runs automated unit and integration tests.

- **HTTPX / FastAPI TestClient**
  - Sends test requests to the API and checks the responses.

- **Git and GitHub**
  - Track project changes and store the repository online.

---

# 1:30–3:00 — Dataset and data sources

## Project coverage

- **18 governorates**
  - The model studies regional differences across Tunisia.

- **Years 2002–2020**
  - The project uses historical harvest-year data.

## Dataset size

- **342 candidate rows**
  - These are all rows in the processed dataset before final filtering.

- **338 supervised observations**
  - These contain a valid target and are used for modelling.

- **131 original columns**
  - The processed dataset contains many agricultural and environmental variables.

- **123 approved predictors**
  - These remain after leakage and metadata filtering.

## What one row means

- **One governorate in one harvest year**
  - Example: Beja in 2015 is one observation.

## Six data sources

- **ONAGRI** — agricultural production, area, and yield.
- **CHIRPS** — rainfall.
- **ERA5** — climate and temperature.
- **MODIS** — vegetation indicators such as NDVI.
- **SoilGrids** — soil properties.
- **OpenLandMap** — additional soil and land information.

## Why several sources are needed

- **Yield depends on many factors together**
  - Rainfall alone cannot explain all regional differences.

---

# 3:00–4:30 — Architecture

## Architecture meaning

- **Architecture is the map of the whole system**
  - It shows the main components and how they connect.

## Data layer

- **Stores the processed dataset and result files**
  - This layer keeps the information used by the model and dashboard.

## Machine-learning layer

- **Handles preparation, features, training, and evaluation**
  - This is where the main prediction logic is implemented.

## Persistence layer

- **Stores the trained model and metadata**
  - It allows the model to be reused without retraining every time.

## Application layer

- **Contains Streamlit and FastAPI**
  - Streamlit is for human users; FastAPI is for other software.

## Integration

- **Dashboard and API use the same saved pipeline**
  - This keeps preprocessing and predictions consistent.

## Architecture versus pipeline

- **Architecture = system map**
  - Shows the main parts of the solution.

- **Pipeline = ordered processing steps**
  - Shows what happens to the data from input to prediction.

---

# 4:30–6:30 — Machine-learning pipeline

- **Load the processed CSV**
  - The workflow begins with the canonical governorate-year dataset.

- **Clean important columns**
  - Governorate, year, and target values are checked and standardised.

- **Sort by governorate and year**
  - Correct order is required before creating historical features.

- **Create historical features**
  - Earlier information from the same governorate is added.

- **Remove leakage columns**
  - Columns that could reveal the current target are excluded.

- **Create `X` and `y`**
  - `X` contains the input features; `y` contains the target yield.

- **Handle numeric missing values**
  - Missing numbers are replaced with the median.

- **Handle categorical missing values**
  - Missing categories are replaced with the most frequent value.

- **One-hot encode categories**
  - Text values are converted into 0/1 numeric columns.

- **Train ExtraTrees**
  - The model learns nonlinear patterns and interactions.

- **Save the complete pipeline**
  - Preprocessing and the model are stored together for consistent use.

## Short pipeline

```text
CSV
→ cleaning
→ historical features
→ leakage removal
→ missing-value handling
→ one-hot encoding
→ ExtraTrees
→ saved pipeline
→ dashboard and API
```

---

# 6:30–8:00 — Historical features and leakage

## Historical features

- **Previous yield**
  - Gives the model information about earlier governorate performance.

- **Yield from two earlier observations**
  - Adds a longer historical view.

- **Three-year and five-year rolling means**
  - Smooth short-term changes and show broader trends.

- **Previous production and cultivated area**
  - Add historical agricultural context.

## Why group by governorate?

- **Each region must use its own history**
  - Beja should not use the previous value from Kef or another governorate.

## Why sort by year?

- **The historical order must be correct**
  - Otherwise the “previous” value could come from the wrong year.

## Why use `shift(1)`?

- **It moves the previous observation into the current row**
  - This prevents the current target from being used as its own feature.

## What is leakage?

- **Leakage means giving the model information that reveals the answer**
  - This can create unrealistically strong performance.

## Removed information

- current-year yield,
- current-year production,
- current-year cultivated area,
- current outcome shares,
- target-related proxy columns.

  - These are too closely related to the target for a fair prediction.

## Honest limitation

- **Historical features were not created separately inside every LOYO fold**
  - A future version should do this for stricter temporal isolation.

---

# 8:00–9:30 — Methodology and validation

## CRISP-DM

- **Business understanding**
  - Define the agricultural problem and prediction objective.

- **Data understanding**
  - Study the years, governorates, missing values, target, and feature groups.

- **Data preparation**
  - Clean, combine, and transform the data.

- **Modelling**
  - Compare regression models and baselines.

- **Evaluation**
  - Measure performance on unseen years.

- **Deployment**
  - Save the model and connect it to the dashboard and API.

## Kanban

- **Task-management method**
  - Used to organise data, modelling, dashboard, API, testing, and documentation work.

## LOYO validation

- **Leave-One-Year-Out**
  - One complete year is held out for validation.

- **Train on all other years**
  - The held-out year is not used during training.

- **Repeat for every year**
  - Every harvest year is tested once.

## Why not a random split?

- **Rows from the same year may share climate conditions**
  - A random split could place similar rows in training and testing.

- **Results could look too optimistic**
  - LOYO gives a more realistic test on a complete unseen year.

---

# 9:30–12:00 — Models, ExtraTrees, and metrics

## Models compared

- **Linear models**
  - Simple references for comparison.

- **Tree models**
  - Learn nonlinear decision rules.

- **Boosting models**
  - Build models gradually to reduce errors.

- **Historical formulas and baselines**
  - Check whether machine learning adds value.

- **Experimental blends**
  - Combine predictions from more than one model.

## ExtraTrees

- **An ensemble of many decision trees**
  - Each tree predicts, and the final result is the average.

- **Works well with tabular data**
  - Tabular means data organised in rows and columns like a spreadsheet.

- **Learns nonlinear relationships**
  - For example, the effect of rainfall can depend on soil, heat, vegetation, or region.

## Main settings

- **800 trees**
  - Makes the average prediction more stable.

- **Minimum two observations per leaf**
  - Reduces rules based on only one row.

- **Square-root feature selection**
  - Each split checks a random subset of predictors, increasing tree diversity.

- **Random seed 42**
  - Makes the result reproducible.

- **All CPU cores**
  - Speeds up training.

## MAE

- **Average prediction mistake**
  - Final MAE: 2.925 qx/ha.
  - Predictions are about 2.9 qx/ha away from the real yield on average.

## RMSE

- **Error metric that gives more weight to large mistakes**
  - Final RMSE: 4.097 qx/ha.
  - Lower is better.

## R²

- **Shows how much of the yield differences the model explains**
  - Final R²: 0.758.
  - The model explains about 75.8% of the observed differences in yield.

- **Not the same as accuracy**
  - It does not mean every prediction is 75.8% correct.

## Why standard ExtraTrees?

- **Performance was close to the best alternatives**
  - A deeper version had slightly lower RMSE and a blend had slightly lower MAE.

- **Simpler to deploy and maintain**
  - Only one model needs to be saved and loaded.

- **Easier to explain**
  - It provides feature importance.

- **Easy to integrate**
  - It works cleanly with Streamlit and FastAPI.

---

# 12:00–13:30 — Explainability and error analysis

## Feature importance

- **Shows which features were useful overall**
  - It gives a global explanation of the model.

- **Does not prove causation**
  - A useful feature is not automatically the cause of yield change.

## Actual versus predicted plots

- **Compare model predictions with real values**
  - Smaller gaps mean better predictions.

## Residual analysis

- **Residual = actual value − predicted value**
  - It shows the size and direction of the error.

## Error by year

- **Shows difficult harvest years**
  - In the saved analysis, 2002 had the highest year-level MAE.

## Error by governorate

- **Shows difficult regions**
  - In the saved analysis, Sidi Bouzid had the highest governorate-level MAE.

## SHAP and LIME

- **Not included in the current project**
  - SHAP would be a useful future improvement for individual explanations.

## Why no confusion matrix?

- **A confusion matrix is for classification**
  - This project predicts a continuous number, so residual analysis is more suitable.

---

# 13:30–15:00 — Testing, versioning, challenge, and future work

## Automated testing

- **10 pytest tests**
  - They verify important preprocessing and API behaviour.

## Unit tests

- **Check individual functions**
  - Examples: data loading, sorting, historical features, and leakage filtering.

## Integration tests

- **Check several components working together**
  - FastAPI, the saved model, metadata, preprocessing, and JSON responses are tested together.

## Manual dashboard testing

- **Check the interface directly**
  - Pages, charts, filters, tables, and single prediction were tested manually.

## Versioning

- **Git tracks code changes**
  - It records the development history.

- **GitHub stores the project online**
  - It supports backup, collaboration, and version control.

## Biggest challenge

- **Combining six different sources while avoiding leakage**
  - The sources had different names, formats, years, missing values, and measurement types.

## First future improvement

- **Create historical features separately inside each validation fold**
  - This would make validation stricter.

## Other improvements

- **Newer data** — the current data ends in 2020.
- **Crop-specific models** — separate models for each cereal crop.
- **Prediction intervals** — show a likely range, not only one number.
- **SHAP** — explain individual predictions.
- **Monitoring** — detect changes in data or model performance.
- **Secure cloud deployment** — add authentication, logging, and monitoring.

---

# Dashboard demonstration — 15:00–24:00

# 15:00–16:00 — Overview page

- **Purpose: summarise the project**
  - Gives the main dataset and performance information quickly.

- **338 supervised observations**
  - Rows used for modelling.

- **18 governorates**
  - Regional coverage.

- **2002–2020**
  - Historical time coverage.

- **123 model inputs**
  - Approved predictors after filtering.

- **RMSE: 4.097**
  - More affected by large errors.

- **MAE: 2.925**
  - Average prediction mistake.

- **R²: 0.758**
  - Explains about 75.8% of observed yield differences.

- **Dataset table**
  - Shows the governorate-year structure of the processed data.

- **Important clarification**
  - The metrics come from LOYO, not from testing the final model on its training rows.

---

# 16:00–17:00 — National View

- **Purpose: create a national summary**
  - Combines governorate-level predictions by year.

- **Area weighting**
  - Governorates with larger cereal areas have more influence.

- **Horizontal axis**
  - Harvest year.

- **Vertical axis**
  - Yield in qx/ha.

- **Actual line**
  - The real national yield.

- **Predicted line**
  - The model-based national estimate.

- **Lines close together**
  - The prediction is close to the actual value.

- **Large gap**
  - The prediction error is larger.

- **Important clarification**
  - This is an aggregation of governorate predictions, not a separate national model.

---

# 17:00–18:00 — Drought Risk Prototype

- **Purpose: show an experimental drought-risk interpretation**
  - Helps identify records with several stress signals.

- **Separate from ExtraTrees**
  - It is a transparent rule-based module, not the yield model.

- **Rainfall deficit**
  - Measures lack of rainfall.

- **Vegetation stress**
  - Shows weaker vegetation conditions.

- **Soil-water stress**
  - Represents soil and water pressure.

- **Heat stress**
  - Represents high-temperature pressure.

- **Risk score from 0 to 100**
  - A higher score means more drought-related stress.

- **Distribution chart**
  - Shows how many governorates are Low, Moderate, or High risk.

- **Ranking chart**
  - Longer bars mean higher governorate risk scores.

- **Trend chart**
  - Shows how the stress components change over time.

- **Important clarification**
  - It is not an official warning system and does not affect RMSE, MAE, or R².

---

# 18:00–19:30 — Model Performance page

- **Purpose: compare models and baselines**
  - Helps justify the final model selection.

- **Comparison table**
  - Shows RMSE, MAE, and R² for each approach.

- **Horizontal RMSE axis**
  - Smaller values are better.

- **Vertical model-name axis**
  - Lists the tested approaches.

- **Shorter bar**
  - Means lower prediction error.

- **ExtraTrees selection**
  - It balanced strong performance with simple deployment and maintenance.

---

# 19:30–20:30 — Feature Importance page

- **Purpose: show the most useful model inputs**
  - Provides a global explanation of the model.

- **Vertical axis**
  - Feature names.

- **Horizontal axis**
  - Importance value.

- **Longer bar**
  - The model used that feature more strongly.

- **Important limitation**
  - Importance is not causation and does not explain one specific prediction.

---

# 20:30–21:30 — Regional Insights page

- **Purpose: show where the model performs better or worse**
  - Overall metrics can hide regional or yearly problems.

- **Year-level analysis**
  - Identifies difficult or unusual harvest years.

- **Governorate MAE chart**
  - Shows the average error for each region.

- **Longer governorate bar**
  - Means a larger average prediction error.

- **Positive bias**
  - The model tends to predict too high.

- **Negative bias**
  - The model tends to predict too low.

- **Important point**
  - This regression error analysis is used instead of a confusion matrix.

---

# 21:30–22:30 — Single Prediction page

- **Purpose: demonstrate one prediction**
  - Shows how the saved pipeline works during deployment.

- **Select a governorate and year**
  - The dashboard loads one historical record.

- **Predicted yield**
  - Model output in qx/ha.

- **Actual yield**
  - Real recorded value.

- **Absolute error**
  - Distance between the prediction and the actual value.

- **Prediction process**

```text
selected row
→ missing-value handling
→ one-hot encoding
→ ExtraTrees
→ predicted yield
```

- **Important clarification**
  - This is a deployment demonstration; official performance comes from LOYO.

---

# 22:30–23:30 — API / Deployment Notes

- **Dashboard is for human users**
  - It presents charts, tables, and controls.

- **API is for other software**
  - Another application can request a prediction automatically.

- **JSON input**
  - Feature values are sent to the API.

- **JSON output**
  - The API returns the predicted yield.

- **`/health`**
  - Checks whether the API, model, and metadata loaded correctly.

- **`/model-info`**
  - Returns information about the selected model.

- **`/features`**
  - Returns the expected feature names.

- **`/sample-input`**
  - Provides an example request.

- **`/predict`**
  - Receives features and returns a prediction.

- **FastAPI documentation**
  - Available at `http://127.0.0.1:8000/docs`.

- **Current deployment**
  - A local academic prototype using Streamlit, FastAPI, and Uvicorn.

- **Production requirements**
  - Authentication, stronger validation, logging, monitoring, cloud hosting, and drift detection.

---

# 23:30–25:00 — Closing

- **End-to-end framework**
  - Covers data, modelling, evaluation, interpretation, testing, and deployment.

- **Six data sources**
  - Gives the model a broader view of agricultural and environmental conditions.

- **Historical features and leakage reduction**
  - Tries to use realistic earlier information.

- **LOYO validation**
  - Tests complete unseen harvest years.

- **ExtraTrees selected**
  - Chosen for strong performance and simple deployment.

- **Final metrics**
  - RMSE: 4.097 qx/ha.
  - MAE: 2.925 qx/ha.
  - R²: 0.758.

- **Streamlit dashboard**
  - Human-facing decision-support interface.

- **FastAPI REST API**
  - Software-facing prediction interface.

- **Final limitation**
  - It is an academic prototype and should be combined with official data and expert judgement.

## Closing sentence

> The project can support agricultural analysis, food planning, and resource-allocation decisions by helping identify difficult years or governorates, but it should not replace official statistics or agronomic expertise.

---

# Keep these for questions

- **Why no accuracy, F1, or ROC-AUC?**
  - The project predicts a continuous yield value, so it is regression.

- **Why no loss curves?**
  - ExtraTrees does not train through epochs like a neural network.

- **Why not a random split?**
  - Same-year climate conditions could appear in both training and testing.

- **How was leakage reduced?**
  - Current-year outcome columns were removed and historical features use earlier observations.

- **Why 123 inputs but more feature-importance rows?**
  - One-hot encoding expands categorical inputs into several columns.

- **Architecture versus pipeline?**
  - Architecture is the system map; pipeline is the ordered processing sequence.

- **Unit tests versus integration tests?**
  - Unit tests check one function; integration tests check several components working together.

- **Main limitation?**
  - Small dataset ending in 2020 and historical features not yet created separately inside each fold.

- **First improvement?**
  - Create historical features inside each LOYO fold for stricter validation.
