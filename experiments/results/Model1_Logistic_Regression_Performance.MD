# Model 1 Performance: Logistic Regression

This document presents the test-set performance of Model 1 (Logistic Regression), described in [Model1_Logistic_Regression.MD](../../src/models/Model1_Logistic_Regression.MD).

## File

**Notebook:** [`src/models/model1_logistic_regression.ipynb`](../../src/models/model1_logistic_regression.ipynb)

## How to Use

1. Ensure `model_features.csv` exists in `data/processed/` (produced by Feature Engineering).
2. Open `model1_logistic_regression.ipynb` and run all cells in order, top to bottom.
3. The metrics and confusion matrix below are produced by the "Evaluate on Test Set" section, using the final model selected in "Final Model Selection" (`C=0.001`).

## Test Set

Evaluation uses the held-out time-based test set (final two quarters, 2017-03 and 2017-06): 5,093 institution-quarter observations, 114 positive (concentration-amplified loss) cases (2.2%).

## Metrics

| Metric | Value |
|---|---|
| ROC-AUC | 0.921 |
| Accuracy | 0.88 |
| Precision (positive class) | 0.13 |
| Recall (positive class) | 0.78 |
| F1-score (positive class) | 0.23 |

## Confusion Matrix

| | Predicted: No Loss | Predicted: Loss |
|---|---|---|
| **Actual: No Loss** | 4,393 | 586 |
| **Actual: Loss** | 25 | 89 |

## Interpretation

**High recall relative to a low base rate, and low precision, reflect a deliberate trade-off made during hyperparameter selection, not an unaddressed weakness.** The model correctly identifies 89 of 114 genuinely at-risk portfolios in the test set, at the cost of 586 false positives. `C=0.001` was selected specifically to produce a correctly-signed, interpretable `hhi` coefficient, at the cost of some recall relative to weaker regularisation (see Model1_Logistic_Regression.MD for the full trade-off analysis). Given this study's purpose, flagging portfolios for STADIOEquities to review before a loss occurs, both the missed cases and the false-positive volume represent genuine, quantified costs.