
where `holding_forward_return` is each holding's price return from the current quarter-end to the next quarter-end, and `benchmark_forward_return` is the S&P 500's return over the same period. This makes concentration mechanically part of the outcome: a portfolio heavily concentrated in a stock that underperforms the market will show a large negative weighted excess return, while the same stock's decline in a diversified portfolio would barely move the result.

**Threshold selection:** the problem statement's methodology originally anticipated a -15% threshold as a starting estimate. This was checked empirically against the actual computed distribution of `weighted_excess_return` (mean 0.01, std 0.05, min -1.04, max 0.79) before being finalised, per the commitment made at the problem-statement stage to verify rather than assume this figure. At -15%, only 0.7% of institution-quarters (134 of 20,244) were flagged positive, too rare to support meaningful model training. Several thresholds were tested:

| Threshold | Positive cases | Proportion |
|---|---|---|
| -15% | 134 | 0.7% |
| -12% | 189 | 0.9% |
| -10% | 283 | 1.4% |
| -7% | 516 | 2.5% |
| **-5%** | **854** | **4.2%** |

**-5% was selected** as the final threshold, balancing a meaningful definition of underperformance (5 percentage points worse than the market, weighted by concentration) against having a sufficient number of positive examples for the models to learn a genuine pattern.

## Key Issues Found and Resolved

| Issue | Finding | Resolution |
|---|---|---|
| NaN poisoning in label calculation | A small number of tickers (e.g. `TEV`) returned `NaN` (not `None`) from price lookups on certain dates, due to gaps in the underlying price series. Since `NaN` arithmetic silently propagates (`NaN + x = NaN`), a single bad ticker within an otherwise-valid 83-holding portfolio caused that entire institution-quarter's weighted excess return to become `NaN`, affecting 1,610 of 20,244 rows (7.9%) before being caught | Explicit `pd.isna()` checks added to both the per-holding and benchmark return functions, alongside a flag confirming at least one valid holding return exists before computing a portfolio's label, rather than defaulting to a misleading `0` |
| Benchmark file structure | `sp500_benchmark.csv`, saved by `yfinance`, uses a two-level column header (`Price`/`Ticker`), which caused a `DateParseError` when read with a naive single-header assumption | Read with `header=[0, 1]`, then simplified to a plain Close-price Series before use |

## Visualisations

Three supporting visualisations are produced in the notebook: the distribution of `weighted_excess_return` with the selected threshold marked, the resulting label's class balance, and a scatter of HHI against excess return, which shows loss cases (label = 1) skewing toward higher concentration, consistent with this study's core hypothesis.

## Limitations

- **Filing-date lag not yet implemented.** As documented in Preprocessing.MD, the SEC's 45-day filing deadline was intended as a proxy for the earliest point information could plausibly have been acted upon. Due to time constraints, this pipeline currently uses the quarter-end date directly as the information cutoff for both feature calculation and label construction, rather than applying the 45-day offset. This introduces a degree of look-ahead bias: a small amount of information (a holding's exact quarter-end position) is treated as known slightly earlier than it would genuinely have been available via the actual 13F filing. This is flagged as a known limitation and a direction for future refinement, rather than a silently-applied assumption.
- **Sector coverage.** Industry sector was successfully sourced for 99% of equity tickers (33 of 3,451 returned no data from `yfinance`, consistent with the same survivorship pattern documented in Preprocessing.MD for delisted or defunct securities).
- **Class imbalance.** The final label is imbalanced (4.2% positive class), consistent with concentration-driven losses being a genuine minority outcome rather than a common one. This must inform metric selection during model evaluation (precision, recall, F1, and ROC/PR-AUC rather than plain accuracy).

## Final Output

`model_features.csv` contains 20,244 institution-quarter observations, with features `hhi`, `top_holding_pct`, `num_holdings`, `sector_hhi`, `total_value`, and the binary target `label`, ready for use in model training.