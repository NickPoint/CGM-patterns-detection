# Machine Learning Approaches for Detecting and Explaining CGM Patterns

## Overview

This project explores machine learning methods to automatically detect clinically relevant glucose patterns from Continuous Glucose Monitoring (CGM) data and translate them into clear, patient-friendly explanations. The goal is to support clinicians in treatment decisions and help people with Type 1 Diabetes (T1D) better understand their glucose dynamics.

## Problem Statement

CGM devices generate high-frequency time-series data reflecting glucose responses to meals, insulin, exercise, and sleep. These patterns are complex, nonlinear, and highly individual, making manual interpretation difficult at scale. Automated pattern detection and explanation can improve safety, clinical insight, and patient self-management.

## Objectives

* Detect key glucose patterns (e.g. hypoglycemia, hyperglycemia, rapid post-meal spikes, stable overnight profiles)
* Compare rule-based detection with machine learning approaches
* Predict future glucose dynamics using sequence models
* Provide interpretable, human-readable explanations of detected patterns

## Data

* **AZT1D Dataset** (25 individuals with T1D, multi-week CGM and pump data)
  Includes CGM readings, insulin delivery, carbohydrate intake, and device modes
  Source: [https://data.mendeley.com/datasets/gk9m674wcx/1](https://data.mendeley.com/datasets/gk9m674wcx/1)
* **DiaTrend**
  The DiaTrend dataset is composed of intensive longitudinal data from wearable medical devices, including a total of 27,561 days of continuous glucose monitor data and 8,220 days of insulin pump data from 54 patients with diabetes, primarily with type 1 diabetes. 
  Source: [https://www.synapse.org/Synapse:syn38187184/wiki/619490](https://www.synapse.org/Synapse:syn38187184/wiki/619490)

## Methodology

* **Preprocessing:** data inspection, missing value handling, normalization, feature extraction
* **Pattern Detection:**

  * Rule-based logic
  * Unsupervised learning (DBSCAN, t-SNE, UMAP)
* **Prediction Models:**

  * Tree-based models (XGBoost)
  * Sequence models (LSTM, GRU)
* **Interpretability:**

  * SHAP and LIME for feature attribution
  * Attention analysis for sequence models
* **Explanations:** rule-based templates or NLP-based summarization

## Evaluation

* **Event Detection:** Precision, Recall, F1-score
* **Glucose Prediction:** MAE, RMSE
* **Clustering:** Silhouette score and clinical pattern alignment
* **Interpretability:** alignment with known clinical knowledge

## Tools & Libraries

* Python, Google Colab
* pandas, numpy, scikit-learn, tsfresh, sktime
* TensorFlow, PyTorch
* XGBoost, LightGBM
* SHAP, LIME
* matplotlib, seaborn

