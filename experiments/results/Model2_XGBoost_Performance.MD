# Model 2 Performance: XGBoost

This document presents the test-set performance of Model 2 (XGBoost), described in [Model2.MD](../../src/models/Model2.MD).

## File

**Notebook:** [`src/models/model2_xgboost.ipynb`](../../src/models/model2_xgboost.ipynb) — see the "Evaluate on Test Set" section, which computes all metrics reported below.

## Test Set

Identical time-based test set used for Model 1: 5,093 institution-quarter observations, 76 positive cases (1.5%), ensuring a directly comparable evaluation.

## Metrics

| Metric | Value |
|---|---|
| ROC-AUC | 0.937 |
| Accuracy | 0.78 |
| Precision (positive class) | 0.06 |
| Recall (positive class) | 0.95 |
| F1-score (positive class) | 0.12 |

## Confusion Matrix

## Confusion Matrix

| | Predicted: No Loss | Predicted: Loss |
|---|---|---|
| **Actual: No Loss** | 3,924 | 1,092 |
| **Actual: Loss** | 4 | 73 |

## Interpretation

**ROC-AUC (0.937) is marginally higher than Model 1's 0.935**, essentially equivalent ranking performance between the two models. Recall (0.95) matches Model 1 exactly, while precision (0.06) is slightly lower, both consequences of `scale_pos_weight` compensating for the same class imbalance.

**The more meaningful difference between the two models is not captured by these headline metrics at all**, it is the feature importance finding: `hhi` is identified as the dominant predictive feature (0.613 importance), correctly reflecting the central role of overall portfolio concentration in this study's hypothesis, whereas Model 1's linear structure produced a confusing, negative `hhi` coefficient due to multicollinearity (see Model2.MD for full discussion).