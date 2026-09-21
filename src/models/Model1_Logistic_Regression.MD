# Model 1: Logistic Regression

This document describes Model 1, a Logistic Regression classifier predicting concentration-driven portfolio risk, used as an interpretable baseline for comparison against Model 2 (XGBoost).

## File

**Notebook:** [`src/models/model1_logistic_regression.ipynb`](model1_logistic_regression.ipynb)
**Dependencies:** [`src/models/requirements.txt`](requirements.txt)
**Input:** `data/processed/model_features.csv` (produced by [Feature Engineering](../features/FeatureEngineering.MD))
**Output:** `artifacts/model1_logistic_regression.pkl`, `artifacts/model1_scaler.pkl`

## How to Use This Notebook

1. Ensure `feature_engineering.ipynb` has been run first, and `model_features.csv` exists in `data/processed/`.
2. Install dependencies: `pip install -r requirements.txt`
3. Open `model1_logistic_regression.ipynb` and run all cells in order, top to bottom.
4. Runs in under a minute; no external API calls or long-running steps.

## Technique

**Logistic Regression** was selected as the interpretable baseline model, chosen specifically for its ability to show, via coefficient signs and magnitudes, which features drive predicted risk, valuable for a client-facing recommendation, where explaining *why* a portfolio is flagged matters as much as the flag itself.

## Data Split

A **time-based split** is used rather than a random split, given the panel (quarterly, repeated-institution) structure of the data: training uses the first 8 quarters (2015-03 to 2016-12), testing uses the final 2 quarters (2017-03, 2017-06). This avoids a model trained partly on later market conditions being evaluated on earlier ones, a form of leakage a random split would risk.

| Split | Rows | Positive Rate |
|---|---|---|
| Train | 15,151 | 4.8% |
| Test | 5,093 | 2.2% |

The test set's lower positive rate (114 positive cases) is a known limitation of time-based splitting on an already-imbalanced label; metrics on the positive class should be interpreted with this sample size in mind.

## Hyperparameters

| Parameter | Value | Reasoning |
|---|---|---|
| `C` | `0.001` | Selected via grid search using `TimeSeriesSplit` cross-validation (5 folds), which respects the chronological ordering of quarters rather than shuffling data randomly. `C=0.001` was chosen because it produces a correctly-signed, interpretable `hhi` coefficient, addressing a multicollinearity issue between `hhi` and `top_holding_pct` (see Feature Coefficients below); ROC-AUC is essentially flat across the tested range of `C` values, so this selection prioritises interpretability without a meaningful cost to ranking performance |
| `class_weight` | `"balanced"` | Compensates for label imbalance (4.2% positive overall) by weighting the minority class more heavily during training, avoiding the need for manual oversampling/undersampling |
| `max_iter` | `1000` | Increased from the default to ensure convergence given standardised, moderate-dimensionality input |
| `random_state` | `42` | Fixed for reproducibility |

**Preprocessing:** all five features (`hhi`, `top_holding_pct`, `num_holdings`, `sector_hhi`, `total_value`) are standardised (zero mean, unit variance) via `StandardScaler`, since Logistic Regression's coefficients and convergence are sensitive to feature scale, and these five features span very different natural ranges (e.g. `total_value` in the hundreds of millions vs. `hhi` between 0 and 1).

## Feature Coefficients

| Feature | Coefficient | Direction |
|---|---|---|
| `top_holding_pct` | +0.55 | Higher single-stock concentration → higher predicted risk |
| `hhi` | +0.29 | Higher overall portfolio concentration → higher predicted risk |
| `sector_hhi` | +0.26 | Higher sector concentration → higher predicted risk |
| `total_value` | +0.04 | Negligible; portfolio size shows little independent effect once concentration is accounted for |
| `num_holdings` | -0.03 | More distinct holdings → lower predicted risk |

**All five coefficients are directionally consistent with this study's hypothesis**, with concentration measures (`top_holding_pct`, `hhi`, `sector_hhi`) all positively associated with predicted risk, and a greater number of distinct holdings associated with lower risk. `hhi` and `top_holding_pct` are correlated at 0.96, since a portfolio's largest holding is the dominant term in its own HHI calculation; achieving a correctly-signed `hhi` coefficient despite this correlation required the regularisation strength documented above.

## Performance Summary

| Metric | Value |
|---|---|
| ROC-AUC | 0.921 |
| Recall (positive class) | 0.78 |
| Precision (positive class) | 0.13 |
| Accuracy | 0.88 |

**High recall relative to a low base rate, and low precision, reflect a deliberate trade-off, not an unaddressed weakness.** The model correctly identifies 89 of 114 genuinely at-risk portfolios in the test set, at the cost of 586 false positives. This trade-off is a direct consequence of prioritising a correctly-signed, interpretable `hhi` coefficient during hyperparameter selection (see Hyperparameters above); a less regularised model recovers additional recall at the cost of `hhi`'s interpretability. Given this project's purpose, flagging portfolios for STADIOEquities to review before a loss occurs, both the missed cases and the false-positive volume represent genuine, quantified costs worth weighing directly against the interpretability gained. A full comparison against Model 2's metrics is presented in [Comparison.MD](../../experiments/results/Comparison.MD) (Part C).

## Visualisations

- `reports/charts/model1_coefficients.png`: feature coefficient magnitudes and directions
- `reports/charts/model1_confusion_matrix.png`: test set confusion matrix

## Limitations

- The positive-class count in the test set (114 cases) means precision/recall figures carry sampling uncertainty; a modest number of additional correct or incorrect predictions could shift these metrics noticeably.
- The choice of `C=0.001` explicitly trades recall for `hhi`'s interpretability; at less aggressive regularisation, recall improves modestly while `hhi`'s coefficient reverts to an uninterpretable, collinearity-driven sign. This is a considered decision rather than an unambiguous optimum, and the trade-off should be revisited if STADIOEquities' priorities favour maximising recall over feature-level interpretability.