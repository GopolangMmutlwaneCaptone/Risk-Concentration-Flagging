# Feature Engineering

This document describes the feature engineering pipeline used to build the modelling dataset from the preprocessed SEC 13F holdings data, computing portfolio concentration features and constructing the binary risk label.

## File

**Notebook:** [`src/features/feature_engineering.ipynb`](feature_engineering.ipynb)
**Dependencies:** [`src/features/requirements.txt`](requirements.txt)
**Input:** `data/processed/cleaned_holdings.csv`, `cusip_ticker_sector_mapping_final.csv`, `equity_close_prices_clean.csv`, `sp500_benchmark.csv` (all produced by [Preprocessing](../preprocessing/Preprocessing.MD))
**Output:** `data/processed/model_features.csv`, the final table used for model training

## How to Use This Notebook

1. Ensure `preprocessing.ipynb` has been run first, and its outputs exist in `data/processed/`.
2. Install dependencies: `pip install -r requirements.txt`
3. Open `feature_engineering.ipynb` and run all cells in order, top to bottom.
4. **Expected runtime:** sourcing industry sector via `yfinance` ticker metadata is the slowest step (roughly 60-90 minutes for ~3,451 tickers, due to one API call per ticker with no batching). A checkpointed results file (`sector_mapping.csv`) is saved, so this step does not need to be repeated once completed.
5. The label-construction step (computing forward returns for every holding in every institution-quarter) is also computationally heavier than the feature calculations, given the nested loop over holdings within each portfolio, and takes approximately 22 minutes to complete.

## What This Pipeline Does

1. **Sources industry sector and industry data** for each equity ticker via `yfinance` ticker metadata (`.info`), since the preprocessing pipeline's `security_type` field classifies instrument type (e.g. Common Stock, REIT), not industry (e.g. Technology, Financials).
2. **Merges** cleaned holdings data with the CUSIP-to-ticker mapping and sourced sector data, restricting to genuine equity holdings with usable price data.
3. **Reshapes to long format**, one row per institution-holding-quarter, retaining only non-zero share holdings.
4. **Computes each holding's dollar value** at each quarter-end, using the closest available trading price on or before that date.
5. **Computes five portfolio-level features per institution per quarter:**
   - **HHI** (Herfindahl-Hirschman Index): sum of squared portfolio weights, a standard measure of concentration
   - **Top-holding percentage**: the largest single position's weight in the portfolio
   - **Number of distinct holdings**
   - **Sector HHI**: HHI computed at the sector level, measuring sector concentration rather than individual-stock concentration
   - **Total portfolio value**: the portfolio's total dollar value at that quarter-end
6. **Constructs the binary risk label**: for each institution-quarter, computes the portfolio's concentration-weighted excess return over the *following* quarter relative to the S&P 500 benchmark, using a 45-day filing-date lag (`decision_date = quarter_end + 45 days`) as the information cutoff to avoid look-ahead bias, then flags the portfolio as high-risk (label = 1) if this weighted excess return falls below a defined threshold.

## Label Construction, in Detail

For each institution's portfolio at each quarter-end, the label is computed as:

*weighted_excess_return = Σ (holding_weight × (holding_forward_return − benchmark_forward_return))*

where `holding_forward_return` is each holding's price return from `decision_date` (quarter-end + 45 days, the point at which that quarter's 13F filing becomes public) to the following quarter's `decision_date`, and `benchmark_forward_return` is the S&P 500's return over the same window. Using `decision_date` rather than the raw quarter-end avoids look-ahead bias, since the exact quarter-end position is not genuinely known to the market until the SEC's 45-day filing deadline. This makes concentration mechanically part of the outcome: a portfolio heavily concentrated in a stock that underperforms the market will show a large negative weighted excess return, while the same stock's decline in a diversified portfolio would barely move the result.

**Threshold selection:** the problem statement's methodology originally anticipated a -15% threshold as a starting estimate. This was checked empirically against the actual computed distribution of `weighted_excess_return` (mean 0.008, std 0.050, min -1.049, max 0.847) before being finalised, per the commitment made at the problem-statement stage to verify rather than assume this figure. At -15%, only 0.6% of institution-quarters (118 of 20,244) were flagged positive, too rare to support meaningful model training. Several thresholds were tested:

| Threshold | Positive cases | Proportion |
|---|---|---|
| -15% | 118 | 0.6% |
| -12% | 192 | 0.9% |
| -10% | 252 | 1.2% |
| -7% | 493 | 2.4% |
| **-5%** | **846** | **4.2%** |

**-5% was selected** as the final threshold, balancing a meaningful definition of underperformance (5 percentage points worse than the market, weighted by concentration) against having a sufficient number of positive examples for the models to learn a genuine pattern.

## Key Issues Found and Resolved

| Issue | Finding | Resolution |
|---|---|---|
| NaN poisoning in label calculation | A small number of tickers (e.g. `TEV`) returned `NaN` (not `None`) from price lookups on certain dates, due to gaps in the underlying price series. Since `NaN` arithmetic silently propagates (`NaN + x = NaN`), a single bad ticker within an otherwise-valid 83-holding portfolio caused that entire institution-quarter's weighted excess return to become `NaN`, affecting 1,610 of 20,244 rows (7.9%) before being caught | Explicit `pd.isna()` checks added to both the per-holding and benchmark return functions, alongside a flag confirming at least one valid holding return exists before computing a portfolio's label, rather than defaulting to a misleading `0` |
| Benchmark file structure | `sp500_benchmark.csv`, saved by `yfinance`, uses a two-level column header (`Price`/`Ticker`), which caused a `DateParseError` when read with a naive single-header assumption | Read with `header=[0, 1]`, then simplified to a plain Close-price Series before use |
| Extreme return outliers after filing-date lag | Introducing the 45-day `decision_date` lag shifted which price observations the return calculation reads, surfacing a bad price entry for ticker RDS: an implausible ~156.5x forward return between the 2015-09 and 2015-12 decision dates. Checking across the full dataset found 933 of 4,621,933 holding-quarter return calculations (0.02%) exceeded ±300%, the most extreme reaching nearly 11,000x, all data artifacts rather than genuine price moves | A price-sanity filter excludes any single-quarter holding return beyond ±300% from the weighted excess-return calculation, treating it as missing rather than a real return |

## Visualisations

Six supporting visualisations are produced in the notebook:

- **Excess return distribution:** the distribution of `weighted_excess_return` with the selected -5% threshold marked
- **Threshold selection:** positive case count at each candidate threshold, showing why -15% was too rare to support model training
- **NaN poisoning fix:** NaN label count before vs. after the explicit `isna()` fix (1,610 to 0)
- **Price-sanity filter fix:** holding-level forward return distribution before vs. after excluding returns beyond ±300%, showing how a handful of data artifacts like RDS (~156.5x) distort the scale until they are removed
- **Label class balance:** the resulting label's distribution (4.2% positive)
- **HHI vs. excess return:** a scatter showing loss cases (label = 1) skewing toward higher concentration, consistent with this study's core hypothesis

## Limitations

- **Filing-date lag applies to the label, not the features.** The 45-day filing-date lag (see Label Construction) is applied when measuring forward returns for the label, but portfolio features (HHI, top-holding %, sector HHI, etc.) still use quarter-end holdings and quarter-end prices directly. This is appropriate, since the features describe the portfolio's actual composition at that date rather than predicting a future outcome, but it means the label and the features it is paired with are not evaluated as of the exact same information cutoff.
- **Sector-adjusted benchmark not implemented.** The label is computed relative to a single, flat S&P 500 benchmark, rather than a sector-specific benchmark, which would more precisely isolate concentration-specific risk from general sector-wide movement. The flat benchmark is a standard, defensible simplification, but a sector-adjusted version remains a valid refinement for future work.
- **Sector coverage.** Industry sector was successfully sourced for 99% of equity tickers (33 of 3,451 returned no data from `yfinance`, consistent with the same survivorship pattern documented in Preprocessing.MD for delisted or defunct securities).
- **Class imbalance.** The final label is imbalanced (4.2% positive class), consistent with concentration-driven losses being a genuine minority outcome rather than a common one. This must inform metric selection during model evaluation (precision, recall, F1, and ROC/PR-AUC rather than plain accuracy).

## Final Output

`model_features.csv` contains 20,244 institution-quarter observations, with features `hhi`, `top_holding_pct`, `num_holdings`, `sector_hhi`, `total_value`, and the binary target `label`, ready for use in model training.