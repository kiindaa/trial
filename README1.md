Use this **25-minute presentation plan**, then keep **5 minutes for questions**.

# 25-minute technical evaluation plan

## 0:00–1:30 — Introduction and problem

### Say

> Good morning. My project is called the Tunisia Cereal Yield Forecasting Framework, or TCYFF.
> It is an end-to-end machine-learning decision-support system that predicts aggregate cereal yield for Tunisian governorates.

> The target combines durum wheat, soft wheat, and barley. The prediction unit is quintals per hectare, written as qx/ha. One quintal is 100 kilograms.

> Cereal yield changes between governorates and years because of rainfall, temperature, vegetation, soil conditions, regional differences, and previous agricultural performance. A national average can also hide local problems.
> Why this matters

These differences can make food planning and agricultural support more difficult.

> Project value

The project supports food-planning and resource-allocation decisions by helping identify governorates and years that may have lower cereal yield or need more attention.

> introduce the solution

The solution is an end-to-end machine-learning system that combines agricultural and environmental data, predicts yield at governorate level, and presents the results through a dashboard and API.

> The main beneficiaries are agricultural analysts, government planners, researchers, agricultural organisations, and decision-makers. Farmers may benefit indirectly through better regional planning.

### Must mention

* project name,
* target,
* unit,
* problem,
* beneficiaries,
* decision-support limitation.

---

## 1:30–3:00 — Dataset and data sources

### Say

> The project covers 18 governorates and harvest years from 2002 to 2020.

> The processed dataset contains 342 candidate rows and 131 columns. After removing records without a valid target, 338 supervised observations remain.

> Each row represents one governorate in one harvest year.

> The final model uses 123 approved predictors.

> The master dataset combines six sources: ONAGRI for agricultural statistics, CHIRPS for rainfall, ERA5 for climate and temperature, MODIS for vegetation, SoilGrids for soil information, and OpenLandMap for additional soil variables.

### Do not spend time on

* detailed source-file formats,
* every dataset column,
* every soil variable.

---

## 3:00–4:30 — Architecture

### Say

> The project uses a modular layered architecture.

> The data layer stores the processed dataset and results.
> The machine-learning layer handles preparation, feature engineering, training, and evaluation.
> The persistence layer stores the saved model and metadata.
> The application layer contains the Streamlit dashboard and FastAPI API.

> The dashboard and API both reuse the same saved model pipeline, so the prediction process stays consistent.

### Simple difference

> Architecture is the map of the whole system. Pipeline is the ordered sequence that transforms data into a prediction.

---

## 4:30–6:30 — Machine-learning pipeline

### Say

> The pipeline starts by loading the processed CSV.

> The data is cleaned and sorted by governorate and year. Historical features are then created, leakage columns are removed, and the data is separated into input features X and target y.

> Numeric missing values are filled with the median. Categorical missing values are filled with the most frequent category, and text categories are converted using one-hot encoding.

> The preprocessing and ExtraTrees model are stored together inside one scikit-learn pipeline.

### Short pipeline

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

## 6:30–8:00 — Historical features and leakage

### Say

> Historical features describe earlier agricultural performance in the same governorate.

> Examples include previous yield, yield from two earlier observations, rolling three-year and five-year means, previous production, and previous cultivated area.

> The data is grouped by governorate because each region must use its own history. It is sorted by year so the sequence is correct.

> The important operation is `shift(1)`. It makes the current row use earlier information instead of its own current target.

> Data leakage means giving the model information that reveals the answer. Current-year yield, production, cultivated area, outcome shares, and target-related proxy columns were therefore removed.

### Honest limitation

> A stricter future improvement would generate the historical features separately inside each validation fold.

---

## 8:00–9:30 — Methodology and validation

### Say

> The project follows CRISP-DM: business understanding, data understanding, data preparation, modelling, evaluation, and deployment.

> Kanban was used separately to organise implementation tasks.

> For evaluation, I used Leave-One-Year-Out validation, or LOYO.

> In every round, one complete harvest year is held out, the model trains on the remaining years, and then predicts the held-out year.

> I did not use a random split because rows from the same year may share similar rainfall and national climate conditions. Mixing the same year between training and testing could make performance look too optimistic.

### Key sentence

> LOYO tests whether the model can generalise to a complete unseen harvest year.

---

## 9:30–12:00 — Models, ExtraTrees, and metrics

### Say

> I compared linear models, tree models, boosting models, historical formulas, simple baselines, and experimental blends.

> The selected deployment model is ExtraTrees Regressor.

> ExtraTrees combines many decision trees. Each tree creates a prediction, and the final prediction is their average.

> It is suitable because it can learn nonlinear relationships and interactions in tabular data.

### Main settings

* 800 trees,
* minimum two observations per leaf,
* square-root feature selection at each split,
* random seed 42,
* all CPU cores.

### Results

> The selected model achieved an RMSE of 4.097 qx/ha, an MAE of 2.925 qx/ha, and an R² of 0.758.

### Explain simply

> MAE is the average prediction mistake, so the model is approximately 2.9 qx/ha away from the real yield on average.

> RMSE also measures error, but it gives more importance to large mistakes.

> R² shows how well the model represents the differences between low-yield and high-yield observations. It does not mean that every prediction is 75.8% correct.

### Why this ExtraTrees version?

> A deeper ExtraTrees version had a slightly lower RMSE, and a blend had a slightly lower MAE. The standard ExtraTrees model was selected because its performance was close to the best while being easier to save, explain, maintain, and deploy.

---

## 12:00–13:30 — Explainability and error analysis

### Say

> I used ExtraTrees feature importance, actual-versus-predicted plots, residual analysis, error by year, and error by governorate.

> Feature importance shows which features were useful to the model globally, but it does not prove causation.

> I did not use SHAP or LIME. SHAP would be a useful future improvement for explaining individual predictions.

> Because this is regression, a confusion matrix is not appropriate. I used residual analysis instead.

> A residual is the difference between the actual and predicted values.

> In the saved results, 2002 had the highest year-level MAE, and Sidi Bouzid had the highest governorate-level MAE.

---

## 13:30–15:00 — Testing, versioning, challenge, future work

### Say

> The project includes 10 automated pytest tests.

> The utility tests act as unit tests because they check individual preparation functions such as data loading, historical feature creation, and leakage filtering.

> The API tests act as integration tests because they check that FastAPI, the saved model, metadata, preprocessing, and prediction response work together.

> The Streamlit dashboard was tested manually.

> Git and GitHub are used for code versioning. The repository also stores the processed data, saved model, metadata, requirements, tests, and evaluation outputs.

> The biggest technical challenge was combining six differently structured data sources while avoiding leakage.

> My first future improvement would be to generate historical features separately inside each validation fold. I would also add newer data, crop-specific models, prediction intervals, SHAP, monitoring, and secure cloud deployment.

---

# Dashboard demonstration — 15:00–24:00

Spend approximately **one minute per page**.

---

## 15:00–16:00 — Overview

### Say

> This page summarises the complete project.

> The cards show 338 supervised observations, 18 governorates, years from 2002 to 2020, and 123 approved model inputs.

> The performance cards show the official LOYO results: RMSE 4.097, MAE 2.925, and R² 0.758.

> The dataset table shows the governorate-year structure and some of the environmental and agricultural variables.

### Important

> These metrics come from LOYO validation, not from testing the final model on the same rows used for training.

---

## 16:00–17:00 — National View

### Say

> This page combines governorate predictions into a national yearly estimate.

> It uses an area-weighted average when cereal-area information is available. This means governorates with larger cereal areas have more influence on the national result.

> The line chart compares actual and predicted national yield.

> The horizontal axis shows the harvest year, and the vertical axis shows yield in qx/ha.

> When the two lines are close, the prediction follows the observed value well. A larger gap means a larger error.

### Important

> This is not a separate national model. It is an aggregation of governorate predictions.

---

## 17:00–18:00 — Drought Risk Prototype

### Say

> This page presents an experimental drought-risk interpretation.

> It is separate from the ExtraTrees model and uses transparent rules based on rainfall deficit, vegetation stress, soil-water stress, and heat stress.

> The score goes from 0 to 100. A higher score represents more drought-related stress.

> The first chart shows how many governorates are classified as Low, Moderate, or High risk.

> The ranking chart shows the governorates with the highest risk scores.

> The line chart shows how the individual stress components change over time.

### Important

> This is not an official warning system and is not used to calculate the model performance metrics.

---

## 18:00–19:30 — Model Performance

### Say

> This page compares the candidate models, baselines, formulas, and blends.

> The table presents RMSE, MAE, and R² for every approach.

> The horizontal bar chart compares RMSE. A shorter bar is better because it means a smaller prediction error.

> Standard ExtraTrees was selected because it gave strong performance with simpler deployment and maintenance.

---

## 19:30–20:30 — Feature Importance

### Say

> This page shows which transformed features were most useful to the ExtraTrees model.

> The horizontal axis shows importance, and the vertical axis lists the feature names.

> A longer bar means the model used that feature more when creating its decisions.

> This is a global explanation. It does not prove causation and does not fully explain one individual prediction.

---

## 20:30–21:30 — Regional Insights

### Say

> This page shows where the model has larger or smaller errors.

> The year-level analysis helps identify difficult harvest years.

> The governorate-level chart shows MAE for every governorate. A longer bar means a larger average prediction error.

> Bias shows direction. Positive bias means overprediction, and negative bias means underprediction.

> This detailed regression analysis replaces the need for a confusion matrix.

---

## 21:30–22:30 — Single Prediction

### Say

> This page demonstrates how the saved pipeline produces one prediction.

> I select a governorate and year. The page shows the predicted yield, actual yield, and absolute error.

> The input record passes through missing-value handling, one-hot encoding, and the ExtraTrees model.

> This demonstrates deployment integration. It is not the official validation experiment because the final model was trained on all supervised observations.

---

## 22:30–23:30 — API / Deployment Notes

### Say

> The dashboard is for human users, while the REST API allows another program to use the model.

> The API accepts feature values in JSON format, passes them through the saved pipeline, and returns a yield prediction as JSON.

> The main endpoints are `/health`, `/model-info`, `/features`, `/sample-input`, and `/predict`.

> FastAPI automatically creates interactive documentation at `127.0.0.1:8000/docs`.

> The current deployment is local. A production system would need authentication, stronger validation, logging, monitoring, and cloud hosting.

---

## 23:30–25:00 — Closing

### Say

> To conclude, this project provides an end-to-end governorate-level cereal-yield forecasting framework for Tunisia.

> It combines six data sources, creates historical and environmental features, reduces direct target leakage, evaluates several models using Leave-One-Year-Out validation, selects an ExtraTrees pipeline, and deploys the result through a dashboard and REST API.

> The selected model achieved an RMSE of 4.097 qx/ha, an MAE of 2.925 qx/ha, and an R² of 0.758.

> The project can support agricultural analysis and help identify difficult years or governorates. However, it remains an academic decision-support prototype and should be used together with official statistics, field observations, and expert judgement.

# Keep these for the 5-minute questions

Do not explain these in detail unless asked:

1. Why no accuracy, F1, ROC-AUC, or confusion matrix?
2. Why no training and validation loss curves?
3. Why not a random split?
4. How exactly was leakage reduced?
5. Why was standard ExtraTrees selected?
6. Why 123 inputs but more transformed importance rows?
7. What is the difference between architecture and pipeline?
8. What is the difference between unit and integration testing?
9. What is the biggest limitation?
10. What would you improve first?

## The most important backup answers

**Why no classification metrics?**

> The model predicts a continuous yield value, so this is regression.

**Why no loss curves?**

> ExtraTrees does not train through epochs like a neural network.

**Main limitation?**

> The dataset is small, ends in 2020, and historical features are not yet generated separately inside each validation fold.

**First improvement?**

> Create historical features inside every LOYO fold for stricter validation.
