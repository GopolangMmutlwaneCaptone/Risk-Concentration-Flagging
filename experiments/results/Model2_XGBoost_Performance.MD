# Model 2 Performance: XGBoost

This document presents the test-set performance of Model 2 (XGBoost), described in [Model2_XGBoost.MD](../../src/models/Model2_XGBoost.MD).

## File

**Notebook:** [`src/models/model2_xgboost.ipynb`](../../src/models/model2_xgboost.ipynb)

## How to Use

1. Ensure `model_features.csv` exists in `data/processed/` (produced by Feature Engineering).
2. Ensure `xgboost` is installed (`pip install -r requirements.txt`).
3. Open `model2_xgboost.ipynb` and run all cells in order, top to bottom.
4. The metrics and confusion matrix below are produced by the "Evaluate on Test Set" section, using the final model selected in "Final Model Selection" (tuned via expanded grid search).

## Test Set

Identical time-based test set used for Model 1: 5,093 institution-quarter observations, 114 positive cases (2.2%), ensuring a directly comparable evaluation.

## Metrics

| Metric | Value |
|---|---|
| ROC-AUC | 0.926 |
| Accuracy | 0.79 |
| Precision (positive class) | 0.09 |
| Recall (positive class) | 0.94 |
| F1-score (positive class) | 0.17 |

## Confusion Matrix

| | Predicted: No Loss | Predicted: Loss |
|---|---|---|
| **Actual: No Loss** | 3,935 | 1,044 |
| **Actual: Loss** | 7 | 107 |

## Interpretation

**Accuracy (0.79) is a misleading headline metric on its own, given the 2.2% class imbalance in the test set**, a model that always predicted "no loss" would score 97.8% accuracy while being entirely useless for this study's purpose.

**ROC-AUC (0.926) is marginally higher than Model 1's 0.921**, essentially equivalent ranking performance. Recall (0.94) is substantially higher than Model 1's 0.78, since Model 2's tree-based structure did not require the same interpretability-vs-recall trade-off Model 1 needed to correct an unstable `hhi` coefficient. `hhi` and `top_holding_pct` jointly account for approximately 77% of feature importance, confirming portfolio concentration as the dominant predictive signal (see Model2_XGBoost.MD for the full feature importance analysis, including its sensitivity to column-subsampling).