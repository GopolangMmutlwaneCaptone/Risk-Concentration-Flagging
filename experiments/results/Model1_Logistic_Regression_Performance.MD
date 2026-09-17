# Model 1 Performance: Logistic Regression

This document presents the test-set performance of Model 1 (Logistic Regression), described in [Model1.MD](../../src/models/Model1.MD).

## File

**Notebook:** [`src/models/model1_logistic_regression.ipynb`](../../src/models/model1_logistic_regression.ipynb) — see the "Evaluate on Test Set" section, which computes all metrics reported below.

## Test Set

Evaluation uses the held-out time-based test set (final two quarters, 2017-03 and 2017-06): 5,093 institution-quarter observations, 76 positive (concentration-amplified loss) cases (1.5%).

## Metrics

| Metric | Value |
|---|---|
| ROC-AUC | 0.935 |
| Accuracy | 0.82 |
| Precision (positive class) | 0.08 |
| Recall (positive class) | 0.95 |
| F1-score (positive class) | 0.14 |

## Confusion Matrix

| | Predicted: No Loss | Predicted: Loss |
|---|---|---|
| **Actual: No Loss** | 4,124 | 892 |
| **Actual: Loss** | 4 | 73 |

## Interpretation

**Accuracy (0.82) is a misleading headline metric given the 1.5% class imbalance in the test set** — a model that always predicted "no loss" would score 98.5% accuracy while being entirely useless for this study's purpose. ROC-AUC (0.935) is the more meaningful summary statistic here, since it measures ranking ability independent of any specific classification threshold, and it confirms the model genuinely separates high-risk from low-risk portfolios well.

**Recall (0.95) and precision (0.08) reflect a deliberate trade-off**, driven by `class_weight="balanced"` (see Model1.MD). The model correctly identifies 73 of 77 true positive cases, missing only 4, at the cost of 892 false positives. Given this study's purpose (flagging portfolios for STADIOEquities to review before a loss occurs), this trade-off is directionally appropriate: the cost of missing a genuinely at-risk client is materially higher than the cost of an unnecessary review.

Full discussion of `hhi`'s coefficient anomaly (attributable to multicollinearity with `top_holding_pct`) is provided in Model1.MD.