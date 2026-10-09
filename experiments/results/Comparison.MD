# Comparison: Model 1 (Logistic Regression) vs. Model 2 (XGBoost)

This document compares the two models' performance on the identical held-out test set, drawing on [Model1_Logistic_Regression_Performance.MD](Model1_Logistic_Regression_Performance.MD) and [Model2_XGBoost_Performance.MD](Model2_XGBoost_Performance.MD).

## File

**Notebooks:** [`src/models/model1_logistic_regression.ipynb`](../../src/models/model1_logistic_regression.ipynb), [`src/models/model2_xgboost.ipynb`](../../src/models/model2_xgboost.ipynb)

## How to Use

Both notebooks must be run first, **Model 1 before Model 2**: `model2_xgboost.ipynb` loads the model and scaler that `model1_logistic_regression.ipynb` saves to `artifacts/`, so that it can run the two significance tests below on both models' predictions. Both train and evaluate on the same held-out test set. This document synthesises their results; it does not require separate code to run.

The figures in this document were produced on Python 3.11 with the package versions recorded in [`src/models/requirements.txt`](../../src/models/requirements.txt) (including xgboost 2.0.3). Re-running on a newer environment gives slightly different Model 2 values (for example a ROC-AUC of 0.924 instead of 0.926) with the same conclusions; see the [Reproducibility](../../README.md#reproducibility) section of the root README.

## Side-by-Side Metrics

| Metric | Model 1 (Logistic Regression) | Model 2 (XGBoost) |
|---|---|---|
| ROC-AUC | 0.921 | **0.926** |
| Accuracy | **0.88** | 0.79 |
| Precision (positive class) | **0.13** | 0.09 |
| Recall (positive class) | 0.78 | **0.94** |
| F1-score (positive class) | **0.23** | 0.17 |

![Model comparison chart](../../reports/charts/model_comparison.png)

## Confusion Matrices

| | Model 1: Pred. No Loss | Model 1: Pred. Loss | Model 2: Pred. No Loss | Model 2: Pred. Loss |
|---|---|---|---|---|
| **Actual: No Loss** | 4,393 | 586 | 3,935 | 1,044 |
| **Actual: Loss** | 25 | 89 | 7 | 107 |

Model 2 catches substantially more true positives (107 vs. 89) at the cost of a higher false-positive rate (1,044 vs. 586).

## Statistical Significance Testing

**McNemar's test** was applied to the two models' binary predictions on the identical test set, testing whether their classification patterns differ significantly:

| | Model 2 Correct | Model 2 Wrong |
|---|---|---|
| **Model 1 Correct** | 4,022 | 460 |
| **Model 1 Wrong** | 20 | 591 |

The result (χ²=401.5, p<0.0001) confirms the two models' classification patterns differ significantly, not attributable to chance.

**DeLong's test** was applied to the two models' predicted probabilities, testing whether their ROC-AUC values differ significantly:

| Model | ROC-AUC |
|---|---|
| Model 1 | 0.9211 |
| Model 2 | 0.9255 |

The result (z=-1.253, p=0.210) finds no statistically significant difference between the two models' ROC-AUC values.

**Together, these results indicate the two models rank risk comparably overall (DeLong's), but differ in which specific cases they classify correctly at a fixed decision threshold (McNemar's).** The disagreements are lopsided: Model 1 is right and Model 2 wrong on 460 test rows, against 20 the other way round, which largely reflects Model 2's higher false-positive count (1,044 vs. 586). McNemar's test compares overall correctness, so it does not test recall. Model 2's advantage is its higher recall on the rare, high-cost positive class (0.94 vs. 0.78), which this test does not evaluate.

## Discussion

**On raw ranking ability, the two models are close.** ROC-AUC differs by only about 0.004 (0.9211 vs. 0.9255). Model 1 achieves higher precision, accuracy, and F1, while Model 2 achieves substantially higher recall.

**This difference is consistent with, among other factors, each model's hyperparameter selection.** Model 1's Logistic Regression suffers from a 0.96 correlation between `hhi` and `top_holding_pct`; achieving a correctly-signed, interpretable `hhi` coefficient required strong regularisation (`C=0.001`), which came at a recall cost. Model 2's tree-based structure handles this same correlation without producing an unstable coefficient, so no equivalent trade-off was needed, and its `scale_pos_weight` setting, which up-weights the positive class, is another likely contributor to its higher recall. These explanations were not isolated experimentally.

**Recommendation:** Model 2 (XGBoost) is preferred. It achieves higher recall, meaningfully more true positives caught, while still producing an interpretable, dominant concentration signal (`hhi` and `top_holding_pct` jointly accounting for ~77% of importance), without requiring Model 1's interpretability-recall trade-off. Given this study's purpose, catching genuinely at-risk portfolios before a loss occurs, Model 2's higher recall is judged more valuable than Model 1's higher precision.

## Limitations of This Comparison

- The test set's positive-class count (114 cases) means metric differences carry some sampling uncertainty; DeLong's test result (p=0.210) is consistent with this, since the ROC-AUC difference between the two models is not statistically significant.
- Neither model's hyperparameter search covered every possible configuration; further tuning may shift these results.
- The held-out test set also informed some modelling choices: Model 1's regularisation trade-off was examined on it, and Model 2's final configuration was chosen partly using test results. The reported test metrics are therefore somewhat optimistic.
- The split of feature importance between `hhi` and `top_holding_pct` varies with the column-sampling setting (see [Model2_XGBoost.MD](../../src/models/Model2_XGBoost.MD)), so the importance ranking should not be over-read.
- Hyperparameter search used cross-validation whose folds are not strictly chronological; see [Known Limitations](../../README.md#known-limitations) in the root README.