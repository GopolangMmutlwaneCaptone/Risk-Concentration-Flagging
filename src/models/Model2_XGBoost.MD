# Model 2: XGBoost

This document describes Model 2, an XGBoost classifier predicting concentration-driven portfolio risk, compared against Model 1 (Logistic Regression) as a more complex, non-linear alternative.

## File

**Notebook:** [`src/models/model2_xgboost.ipynb`](model2_xgboost.ipynb)
**Dependencies:** [`src/models/requirements.txt`](requirements.txt)
**Input:** `data/processed/model_features.csv` (produced by [Feature Engineering](../features/FeatureEngineering.MD))
**Output:** `artifacts/model2_xgboost.pkl`

## How to Use This Notebook

1. Ensure `feature_engineering.ipynb` has been run first, and `model_features.csv` exists in `data/processed/`.
2. Install dependencies, including `xgboost` (`pip install -r requirements.txt`).
3. Open `model2_xgboost.ipynb` and run all cells in order, top to bottom.
4. Runs in under a minute; no external API calls or long-running steps.

## Technique

**XGBoost**, a gradient-boosted decision tree ensemble, was selected as the second model both for its typically strong performance on tabular data with class imbalance, and specifically because it is far more robust to correlated features than a linear model, directly relevant given the multicollinearity between `hhi` and `top_holding_pct` addressed in Model 1 (see [Model1_Logistic_Regression.MD](Model1_Logistic_Regression.MD)). Chen et al. (2023), cited in this project's literature review, also compared tree-ensemble methods (Random Forest, XGBoost, CatBoost) on 13F-derived data, providing direct precedent for this choice.

## Data Split

The identical time-based split used in Model 1 is applied here, ensuring a fair, directly comparable evaluation between the two models.

| Split | Rows | Positive Rate |
|---|---|---|
| Train | 15,151 | 4.8% |
| Test | 5,093 | 2.2% |

## Hyperparameters

| Parameter | Value | Reasoning |
|---|---|---|
| `max_depth` | 3 | Selected via grid search using `TimeSeriesSplit` cross-validation (5 folds) |
| `learning_rate` | 0.01 | Selected via the same grid search |
| `n_estimators` | 50 | Selected via the same grid search |
| `subsample` | 0.7 | Selected via the same grid search; each tree trains on 70% of rows, reducing overfitting risk |
| `colsample_bytree` | 0.7 | Selected via the same grid search; each tree considers 70% of features when splitting |
| `scale_pos_weight` | 19.7 (computed as the train set's negative-to-positive class ratio) | XGBoost's standard mechanism for class imbalance, analogous to Logistic Regression's `class_weight="balanced"` in Model 1 |
| `random_state` | 42 | Fixed for reproducibility |

**Preprocessing:** no feature scaling is applied. Unlike Logistic Regression, tree-based models split on raw feature values at learned thresholds and are insensitive to feature scale, so `StandardScaler` (used in Model 1) is unnecessary here.

**Tuning outcome.** An initial grid search across `max_depth`, `learning_rate`, and `n_estimators` produced a model performing essentially identically to a standard, untuned configuration. A second, more thorough search additionally tuned `subsample` and `colsample_bytree`, selecting the configuration above, which achieves a marginally higher test-set ROC-AUC (0.926) and fewer false positives (1,044) than the first search's result. The configuration from this second, more complete search is retained as the final model.

## Feature Importance

| Feature | Importance |
|---|---|
| `hhi` | 0.390 |
| `top_holding_pct` | 0.380 |
| `sector_hhi` | 0.181 |
| `num_holdings` | 0.028 |
| `total_value` | 0.021 |

**`hhi` and `top_holding_pct` jointly account for approximately 77% of total feature importance at the selected hyperparameters**, confirming portfolio concentration, at both the overall-portfolio and single-holding level, as the dominant predictive signal. The near-even split between these two specific features is a consequence of column subsampling (`colsample_bytree=0.7`, see Hyperparameters above): since the two features are correlated at 0.96 (confirmed via direct correlation analysis), trees trained without access to one naturally rely on the other. This was tested directly by varying `colsample_bytree` from 0.5 to 1.0: individual importance for `hhi` ranges from 0.34 to 0.74 across this range, with `top_holding_pct` moving inversely, while their combined importance is meaningfully more stable, though not fully constant, ranging from 0.62 to 0.86 (see `reports/charts/colsample_stability_check.png`). This confirms concentration as a robust, dominant signal overall, while showing that the precise individual split between these two correlated features is sensitive to this specific hyperparameter and should not be over-interpreted in isolation. This is directionally consistent with this study's core hypothesis and with the concentration-index-based findings in Kacperczyk, Sialm and Zheng (2005) and Yang et al. (2021), both cited in this project's literature review.

## Performance Summary

| Metric | Model 2 (XGBoost) | Model 1 (Logistic Regression) |
|---|---|---|
| ROC-AUC | 0.926 | 0.921 |
| Recall (positive class) | 0.94 | 0.78 |
| Precision (positive class) | 0.09 | 0.13 |
| Accuracy | 0.79 | 0.88 |

**Overall ranking ability (ROC-AUC) is essentially equivalent between the two models.** Model 2 catches a substantially higher proportion of genuinely at-risk portfolios (107 of 114, recall 0.94) than Model 1 (89 of 114, recall 0.78), at the cost of a higher false-positive rate (1,044 vs. 586) and lower accuracy. This difference reflects each model's hyperparameter selection: Model 1's `C` was deliberately chosen to prioritise a correctly-signed, interpretable `hhi` coefficient at a real recall cost, while Model 2 required no equivalent trade-off, since its tree-based structure handles the same underlying correlation without producing an unstable or counter-intuitive coefficient. Model 2's principal advantage over Model 1 is therefore twofold: a more correct and interpretable picture of which features drive risk from the outset, and higher recall without needing to sacrifice that interpretability. A full, formal comparison, including a side-by-side chart of both models' metrics, is presented in [Comparison.MD](../../experiments/results/Comparison.MD) (Part C).

## Visualisations

- `reports/charts/model2_feature_importance.png`: feature importance ranking
- `reports/charts/model2_confusion_matrix.png`: test set confusion matrix
- `reports/charts/model_comparison.png`: side-by-side comparison of Model 1 and Model 2 test-set metrics
- `reports/charts/colsample_stability_check.png`: `hhi` vs. `top_holding_pct` importance across a range of column subsampling rates, demonstrating the stability of their combined importance relative to either individual value

## Limitations

- The positive-class count in the test set (114 cases) means precision/recall figures carry sampling uncertainty.
- Feature importance for `hhi` and `top_holding_pct` should be interpreted with caution: their combined importance is meaningfully more stable than either individual value across tested `colsample_bytree` settings (0.62-0.86), but is not fully constant, and the individual split between the two is highly sensitive to this hyperparameter (see Feature Importance above).