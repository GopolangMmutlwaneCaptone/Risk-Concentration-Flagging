# Preprocessing

This document describes the preprocessing pipeline used to prepare the SEC 13F-HR Institutional Investment Data (the public proxy dataset described in the project's literature review) for feature engineering and modelling.

## File

**Notebook:** [`src/preprocessing/preprocessing.ipynb`](preprocessing.ipynb)
**Dependencies:** [`src/preprocessing/requirements.txt`](requirements.txt)

## How to Use This Notebook

1. Ensure the raw dataset files (`13Fdata.csv`, `institutions.csv`, `stock_names.csv`) are placed in `data/raw/`.
2. Install dependencies: `pip install -r requirements.txt`
3. Open `preprocessing.ipynb` and run all cells in order, top to bottom.
4. **Expected runtime:** the CUSIP-to-ticker mapping step (via the OpenFIGI API) is the most time-consuming part of this pipeline, and can take several hours on a first run, due to the API's rate limits (10 CUSIPs per request, 25 requests per 60 seconds on the free tier) and observed network instability at this scale. The notebook checkpoints progress throughout, so an interrupted run can be resumed without starting over. On subsequent runs, if the mapping and price-download output files already exist in `data/processed/`, those steps can be skipped by loading the existing files directly rather than re-running the API calls.
5. Outputs are saved to `data/processed/`, including the final `equity_close_prices.csv`, the file that Feature Engineering builds on.

## What This Pipeline Does

1. **Loads** the raw 13F holdings dataset.
2. **Restricts** the data to the 11 columns that correspond to genuine 13F quarterly reporting periods (the raw file provides 37 monthly columns, most of which are forward-filled repeats of the same underlying quarterly filing).
3. **Runs a systematic data quality check** (data types, missing values, duplicate records, invalid values, outliers, identifier consistency), rather than relying on a visual inspection alone.
4. **Removes placeholder/invalid CUSIPs** (e.g. `"000000000"`, `"000000ADR"`, `"000000COM"`) that carry no genuine holding information.
5. **Drops the `2015-03` reporting period**, which was found to be entirely zero-valued across all 1,713,673 rows, a corrupted or unpopulated column in the source data, not a usable quarter.
6. **Aggregates genuine duplicate holdings** (the same institution reporting the same security across multiple rows, most likely reflecting multiple sub-accounts or managed portfolios under one parent institution) by summing share counts, after confirming this does not conflate genuinely conflicting reports.
7. **Cleans institution and stock name formatting** (trailing slashes, excess whitespace, multiply-encoded HTML entities).
8. **Maps each CUSIP to a ticker and security type** via the OpenFIGI API, since the raw data provides only CUSIP, a regulatory identifier, not a tradeable ticker symbol. A retry pass is run on initial failures to rule out network-related causes before accepting them as genuine gaps.
9. **Filters to genuine equity instruments** (Common Stock, Depositary Receipt, REIT, Preferred Stock), excluding mutual funds, bonds, and structured debt products, which are not the unit of analysis this study measures.
10. **Downloads historical price data** for all mapped equity tickers, plus the S&P 500 benchmark, via `yfinance`.
11. **Consolidates and filters** the downloaded prices into the final output, `equity_close_prices.csv`.

## Key Data Quality Issues Found and Resolved

| Issue | Finding | Resolution |
|---|---|---|
| CUSIP formatting inconsistency | Leading zeros were silently stripped by pandas on CSV reload unless the column dtype was explicitly forced to string, causing the same security (e.g. AAON Inc, CUSIP `360206`) to appear as two separate, differently-formatted rows | A `standardize_cusip()` function is applied immediately after every CSV load and before every merge/index operation throughout the notebook |
| Placeholder CUSIPs | 461 rows carried non-genuine CUSIP values (e.g. `"000000000"`, `"000000ADR"`, `"000000COM"`), each with zero shares recorded in every quarter | Removed via pattern matching (CUSIP consisting of leading zeros followed only by letters) |
| Unusable reporting period | The `2015-03` column was found to be 100% zero-valued across the entire dataset | Dropped from the usable quarter-end columns (11 → 10 quarters) |
| Duplicate holdings | 82,466 `(cusip, cik)` pairs appeared as duplicate rows after removing placeholders. Investigation of a real example (AGL Resources under Royal Bank of Canada) showed genuine, differing share counts split across rows, not identical repeats | Duplicate rows aggregated by summing share counts per `(cusip, cik)` pair, after confirming this does not conflate genuinely conflicting values (see Assumptions below) |
| Institution/stock name formatting | Trailing slashes/backslashes, excess whitespace, and multiply-encoded HTML entities (e.g. `"Bell State Bank &amp;amp;amp; Trust"`) | Cleaned via a dedicated name-cleaning function |
| CUSIP-to-ticker match ambiguity | A single CUSIP can return many possible OpenFIGI matches across different exchanges and currencies; naively taking the first result risked selecting a non-primary listing (e.g. ABB Ltd's first match was a German exchange ADR, not its actual US-tradeable ticker, `ABBNY`) | A selection function prioritises US-listed common stock, then any US listing, then falls back to the first available result |
| Field naming | OpenFIGI's `securityType2` field (e.g. "Common Stock", "REIT") was initially stored under the name `sector`, which is misleading, since it classifies instrument type, not industry (e.g. Technology, Financials) | Renamed to `security_type` throughout. Genuine industry sector is not provided by this mapping and must be sourced separately in Feature Engineering (e.g. via a GICS mapping or `yfinance` ticker metadata) |

## Limitations

- **CUSIP mapping coverage:** approximately two-thirds of unique CUSIPs could not be resolved to a ticker via OpenFIGI, most commonly because the security was delisted, acquired, or otherwise absent from OpenFIGI's identifier database (confirmed via a retry pass, which recovered only a small fraction of the original failures, ruling out network instability as the primary cause). However, unmapped securities were found to represent a disproportionately smaller share of total holding volume than mapped securities, suggesting they are predominantly minor positions rather than dominant holdings.
- **Price data survivorship bias:** of the 5,997 mapped equity tickers, **2,546 (42.5%) returned no usable price data** from `yfinance`. This is a meaningful limitation, not a routine data gap: securities that were delisted, went bankrupt, or were acquired under financial distress are simultaneously more likely to fail a price download *and* more likely to represent the severe-loss cases this study's label is designed to detect. This exclusion may therefore not be random with respect to outcome, and could bias the resulting model toward underestimating concentration risk. This should be treated as an explicit limitation when interpreting model results, not as missing data that can be safely ignored.
- **No filing-date field:** the raw dataset provides only the reporting period (quarter-end date), not the actual date each 13F was filed. Since institutions have up to 45 days after quarter-end to file, using the quarter-end date directly as the point of "known information" risks a degree of look-ahead bias. In the absence of a true filing date in this dataset, the SEC's 45-day filing deadline is used as a conservative proxy for the earliest point this information could plausibly have been acted upon, a documented assumption rather than a precise measurement.
- **Industry sector not yet sourced:** this pipeline resolves each holding to a *security type* (used to filter to equity instruments), not an *industry sector*. Industry sector, required for this study's sector-concentration feature, must be sourced separately in Feature Engineering.

## Assumptions

- Duplicate `(cusip, cik)` rows represent genuine multiple sub-account or managed-portfolio positions under one parent institution, and are correctly combined by summing, rather than representing erroneous duplicate filings. This is considered the more likely interpretation given that duplicate groups frequently contained several distinct, plausible non-zero values (in one case, six different values) rather than a single repeated figure, but is not certain.
- SEC Form 13F institutional holdings data reasonably approximates portfolio concentration behaviour relevant to STADIOEquities' retail investor risk problem, despite differences in investor type and scale (as previously documented in the project's RAAIDD log).