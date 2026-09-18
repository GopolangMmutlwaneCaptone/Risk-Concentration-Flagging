# Model 2: XGBoost

This document describes Model 2, an XGBoost classifier predicting concentration-driven portfolio risk, compared against Model 1 (Logistic Regression) as a more complex, non-linear alternative.

## File

**Notebook:** [`src/models/model2_xgboost.ipynb`](model2_xgboost.ipynb)
**Input:** `data/processed/model_features.csv` (produced by [Feature Engineering](../features/FeatureEngineering.MD))
**Output:** `artifacts/model2_xgboost.pkl`

## How to Use This Notebook

1. Ensure `feature_engineering.ipynb` has been run first, and `model_features.csv` exists in `data/processed/`.
2. Install `xgboost` if not already available (`pip install -r requirements.txt`).
3. Open `model2_xgboost.ipynb` and run all cells in order, top to bottom.
4. Runs in under a minute; no external API calls or long-running steps.

## Technique

**XGBoost**, a gradient-boosted decision tree ensemble, was selected as the second model both for its typically strong performance on tabular data with class imbalance, and specifically because it is far more robust to correlated features than a linear model, directly relevant given the multicollinearity identified between `hhi` and `top_holding_pct` in Model 1 (see [Model1_Logistic_Regression.MD](Model1_Logistic_Regression.MD)). Chen et al. (2023), cited in this project's literature review, also compared tree-ensemble methods (Random Forest, XGBoost, CatBoost) on 13F-derived data, providing direct precedent for this choice.

## Data Split

The identical time-based split used in Model 1 is applied here, ensuring a fair, directly comparable evaluation between the two models.

| Split | Rows | Positive Rate |
|---|---|---|
| Train | 15,151 | 5.1% |
| Test | 5,093 | 1.5% |

## Hyperparameters

| Parameter | Value | Reasoning |
|---|---|---|
| `n_estimators` | 100 | Standard starting point; not extensively tuned given time constraints (see Limitations) |
| `max_depth` | 4 | Limits tree complexity to reduce overfitting risk on a moderate-sized dataset |
| `learning_rate` | 0.1 | Standard default, balancing convergence speed and stability |
| `scale_pos_weight` | 18.5 (computed as the train set's negative-to-positive class ratio) | XGBoost's standard mechanism for class imbalance, analogous to Logistic Regression's `class_weight="balanced"` in Model 1 |
| `random_state` | 42 | Fixed for reproducibility |

**Preprocessing:** no feature scaling is applied. Unlike Logistic Regression, tree-based models split on raw feature values at learned thresholds and are insensitive to feature scale, so `StandardScaler` (used in Model 1) is unnecessary here.

## Feature Importance

| Feature | Importance |
|---|---|
| `hhi` | 0.613 |
| `sector_hhi` | 0.163 |
| `total_value` | 0.086 |
| `top_holding_pct` | 0.079 |
| `num_holdings` | 0.060 |

**`hhi` is by far the most important feature**, directly resolving the ambiguity found in Model 1, where `hhi`'s coefficient was negative due to multicollinearity with `top_holding_pct`. Freed from the linear-model constraint of assigning one direction per feature, XGBoost correctly identifies overall portfolio concentration (`hhi`) as the dominant predictive signal, consistent with this study's core hypothesis and with the concentration-index-based findings in Kacperczyk, Sialm and Zheng (2005) and Yang et al. (2021), both cited in this project's literature review.

## Performance Summary

| Metric | Model 2 (XGBoost) | Model 1 (Logistic Regression) |
|---|---|---|
| ROC-AUC | 0.937 | 0.935 |
| Recall (positive class) | 0.95 | 0.95 |
| Precision (positive class) | 0.06 | 0.08 |
| Accuracy | 0.78 | 0.82 |

**Overall ranking ability (ROC-AUC) is nearly identical between the two models.** Both correctly prioritise recall over precision as a result of their respective imbalance-correction mechanisms, catching the large majority of genuinely at-risk portfolios at the cost of a high false-positive rate, consistent with the reasoning in Model1_Logistic_Regression.MD (missing genuine risk is more costly than an unnecessary review, given this project's purpose). Model 2's main advantage over Model 1 is not raw predictive performance, but a more correct and interpretable picture of which features actually drive risk (`hhi` as the dominant signal, rather than the confusing, collinearity-driven result in Model 1). A full, formal comparison, including a side-by-side chart of both models' metrics, is presented in [Comparison.MD](../../experiments/results/Comparison.MD) (Part C).

## Visualisations

- `reports/model2_feature_importance.png`: feature importance ranking
- `reports/model2_confusion_matrix.png`: test set confusion matrix
- `reports/model_comparison.png`: side-by-side comparison of Model 1 and Model 2 test-set metrics (generated at the end of this notebook)

## Limitations

- Hyperparameters were set to reasonable standard values rather than tuned via cross-validation or grid search, due to time constraints. A tuned model may achieve improved precision without sacrificing the current recall level.
- As with Model 1, the small number of positive cases in the test set (76) means precision/recall figures carry meaningful sampling uncertainty.