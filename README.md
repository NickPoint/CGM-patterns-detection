# CGM Pattern Detection and Explanation

Prototype pipeline that detects clinically relevant glucose patterns in Continuous Glucose Monitoring (CGM) and insulin pump data, forecasts near-term glucose, and turns both into plain-language summaries for patients and clinicians.

Built on the [DiaTrend](https://www.synapse.org/Synapse:syn38187184/wiki/619490) dataset (54 subjects with Type 1 Diabetes, CGM + insulin pump records). The [AZT1D](https://data.mendeley.com/datasets/gk9m674wcx/1) dataset was used in the early exploration phase.

## Motivation

CGM devices produce a glucose reading every 5 minutes, alongside irregular bolus, basal and carbohydrate events. The resulting time series is continuous, highly individual, and impractical to read manually at scale. The project asks whether the clinically meaningful parts of it can be extracted automatically and expressed in a sentence a patient can act on.

## Pipeline

1. **Preprocessing** — per-subject loading of CGM, bolus and basal sheets; timestamps floored to 5-minute bins, duplicates averaged, streams aligned onto a common index.
2. **Daily metrics** — Time in Range (70–180 mg/dL), Time Below/Above Range, mean, SD and coefficient of variation per subject-day.
3. **Pattern detection** — rule-based detectors grounded in clinical literature (below).
4. **Forecasting** — XGBoost regression on lagged features, 60 minutes ahead.
5. **Risk classification** — XGBoost classifiers for upcoming hypo-/hyperglycaemia (see *Known issues*).
6. **Insight generation** — aggregated pattern counts and risk scores rendered as patient-facing messages.

## Pattern detectors

| Pattern | Rule |
|---|---|
| Nocturnal hypoglycaemia | Glucose < 70 mg/dL between 00:00 and 06:00 |
| Post-prandial hyperglycaemia | Glucose > 180 mg/dL within 2 h of carbohydrate intake |
| Dawn phenomenon | Sustained glucose rise between 03:00 and 07:00 without carb input |
| Possible missed bolus | Carb intake > 10 g with no bolus insulin in the following hour |

The rule set was defined together with the project client, who identified which glucose behaviours are clinically relevant and how each should be expressed. The thresholds follow published clinical criteria.

## Forecasting

Features: 12 lags (60 minutes) each of CGM, carbohydrate input, bolus and basal rate - 48 features per sample. Target is glucose reading 60 minutes ahead. A global model is trained across subjects with one-hot subject IDs. train/test splits are separated by a 7-day temporal gap to limit leakage.

Per-subject results of the global model:

| Metric | Mean across subjects |
|---|---|
| MAE | 3.84 mg/dL |
| MSE | 41.06 mg/dL² |

Per-subject fine-tuning of the global model was tested for personalisation and changed error by less than ±1% in either direction, so the global model was kept.

## What didn't work

**Unsupervised pattern discovery.** UMAP + DBSCAN, Gaussian Mixture Models and k-means were all tried on engineered day-level features. GMM was preferred among them - it works in the original feature space, assumes smooth transitions between latent states, and gives probabilistic assignments, which fits the continuous nature of glucose dynamics. But the clusters separated subjects and days rather than behaviours, feature selection proved decisive and unstable, and "you are in cluster 7" is not an explanation anyone can act on. Thus, clustering wasn't useful for discovering new patterns in the data nor for describing it to a user.

**Risk classification.** The hypo- and hyperglycaemia classifiers report AUC ≈ 0.998, which is not credible. The most likely cause is leakage through overlapping look-ahead windows between train and test. AUC was chosen over accuracy because the positive class is rare.

## Example output

Generated for a single subject over their full record:

```
- You had nocturnal hypoglycemia 336 times. Discuss overnight basal settings with your care team.
- Your glucose rose above 180 mg/dL after meals 175 times. Consider reviewing carb counting or mealtime insulin.
- A dawn phenomenon was observed 943 times. You may need to adjust basal rates overnight.
- There were 2 meals with carbs but no bolus insulin. Make sure to bolus for meals.
```

Messages are deliberately conservative and are intended for clinician review before being shown to a patient.

## Repository

| File | Contents |
|---|---|
| `CGM_project_final.ipynb` | End-to-end pipeline: preprocessing, metrics, detectors, forecasting, insights |
| `CGM_project_sandbox.ipynb` | Exploration: AZT1D inspection, feature engineering, UMAP/DBSCAN and GMM clustering experiments |

## Running it

```bash
pip install pandas numpy scikit-learn xgboost umap-learn openpyxl matplotlib seaborn
```

Download DiaTrend and place the per-subject `.xlsx` files in a `diatrend/` folder next to the notebook, then run `CGM_project_final.ipynb` top to bottom. Notebooks were developed in Google Colab.
