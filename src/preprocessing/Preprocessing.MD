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
5. Outputs are saved to `data/processed/`, including the final `equity_close_prices_clean.csv`, the file that Feature Engineering builds on.

## What This Pipeline Does

1. **Loads** the raw 13F holdings dataset.
2. **Forward-fills monthly share values.** The raw dataset reports a non-zero value only in the specific month a new filing was observed, with `0` recorded in all other months by default, not because the holding was genuinely zero. Forward-filling resolves this into each security's actual, continuous holding history (see Key Data Quality Issues below).
3. **Restricts** the data to the 11 columns that correspond to genuine 13F quarterly reporting periods.
4. **Runs a systematic data quality check** (data types, missing values, duplicate records, invalid values, outliers, identifier consistency), rather than relying on a visual inspection alone.
5. **Removes placeholder/invalid CUSIPs** (e.g. `"000000000"`, `"000000ADR"`, `"000000COM"`, `"000000nan"`) that carry no genuine holding information.
6. **Aggregates duplicate holdings**, distinguishing true duplicate filings from genuine multiple sub-account positions (see Key Data Quality Issues below).
7. **Cleans institution and stock name formatting** (trailing slashes, excess whitespace, multiply-encoded HTML entities).
8. **Maps each CUSIP to a ticker and security type** via the OpenFIGI API, since the raw data provides only CUSIP, a regulatory identifier, not a tradeable ticker symbol. A retry pass is run on initial failures to rule out network-related causes before accepting them as genuine gaps.
9. **Filters to genuine equity instruments** (Common Stock, Depositary Receipt, REIT, Preferred Stock), excluding mutual funds, bonds, and structured debt products, which are not the unit of analysis this study measures.
10. **Downloads historical price data** for all mapped equity tickers, plus the S&P 500 benchmark, via `yfinance`.
11. **Consolidates** the downloaded prices, then removes tickers with no usable price data at all (see Limitations), producing the final output, `equity_close_prices_clean.csv`.

## Key Data Quality Issues Found and Resolved

| Issue | Finding | Resolution |
|---|---|---|
| CUSIP formatting inconsistency | Leading zeros were silently stripped by pandas on CSV reload unless the column dtype was explicitly forced to string, causing the same security (e.g. AAON Inc, CUSIP `360206`) to appear as two separate, differently-formatted rows | A `standardize_cusip()` function is applied immediately after every CSV load and before every merge/index operation throughout the notebook |
| **Missing forward-fill of monthly values** | An earlier version of this pipeline restricted directly to quarter-end columns without forward-filling first, incorrectly treating ~94% of genuine holdings data as zero. Confirmed via direct inspection: e.g. CUSIP `000304105` under institution "Scout Investment Advisors" showed real, non-zero share counts only in scattered non-quarter-end months, appearing as `0` at every quarter-end despite being a continuously held position | All monthly values are forward-filled (each reported value carried forward until superseded by the next reported value) before restricting to quarter-end columns |
| Placeholder CUSIPs | Rows carried non-genuine CUSIP values in three distinct patterns: `"000000000"`, generic filler codes such as `"000000ADR"`/`"000000COM"` shared across unrelated companies, and `"000000nan"` (a stringified missing value). Each carried zero shares in every quarter | Removed via three successive pattern-matching passes; the "nan" pattern uses an anchored match (`^0+nan$`) rather than a substring search, since some genuine CUSIPs (e.g. `29250NAN5`) legitimately contain "NAN" as part of a valid identifier and must not be removed |
| Duplicate holdings, and a double-counting risk in the initial fix | 82,464 `(cusip, cik)` pairs appeared as duplicate rows after placeholder removal. An initial approach summed all duplicate share counts uniformly. Further investigation of conflicting values found this was only correct for roughly half of cases: of 153,494 conflicting duplicate-group-quarters, 79,927 contained *identical* repeated values (true duplicate filings, which summing would double-count) and 73,567 contained *genuinely different* values (consistent with distinct sub-account positions, which summing correctly combines) | Aggregation logic revised to distinguish the two cases: identical non-zero values within a group are treated as one position (deduplicated, not summed); genuinely differing values are summed as separate positions. Verified against a named example: AGL Resources under Royal Bank of Canada's CIK showed 2,044 shares under the initial (incorrect) uniform-summing approach; the corrected approach shows 1,022, confirming two duplicate rows with the identical value 1,022 were previously being double-counted |
| Institution/stock name formatting | Trailing slashes/backslashes, excess whitespace, and multiply-encoded HTML entities (e.g. `"Bell State Bank &amp;amp;amp; Trust"`) | Cleaned via a dedicated name-cleaning function |
| CUSIP-to-ticker match ambiguity | A single CUSIP can return many possible OpenFIGI matches across different exchanges and currencies; naively taking the first result risked selecting a non-primary listing (e.g. ABB Ltd's first match was a German exchange ADR, not its actual US-tradeable ticker, `ABBNY`) | A selection function prioritises US-listed common stock, then any US listing, then falls back to the first available result |
| Field naming | OpenFIGI's `securityType2` field (e.g. "Common Stock", "REIT") was initially stored under the name `sector`, which is misleading, since it classifies instrument type, not industry (e.g. Technology, Financials) | Renamed to `security_type` throughout. Genuine industry sector is not provided by this mapping and must be sourced separately in Feature Engineering (e.g. via a GICS mapping or `yfinance` ticker metadata) |

## Visualisations

Three supporting visualisations are produced in the notebook: CUSIP-to-ticker mapping outcome (mapped vs. unmapped), the distribution of retained equity holdings by security type, and price data availability (illustrating the survivorship bias limitation discussed below).

## Limitations

- **CUSIP mapping coverage:** approximately two-thirds of unique CUSIPs could not be resolved to a ticker via OpenFIGI, most commonly because the security was delisted, acquired, or otherwise absent from OpenFIGI's identifier database (confirmed via a retry pass, which recovered only a small fraction of the original failures, ruling out network instability as the primary cause). However, unmapped securities were found to represent a disproportionately smaller share of total holding volume than mapped securities, suggesting they are predominantly minor positions rather than dominant holdings.
- **Price data survivorship bias:** of the 5,997 mapped equity tickers, **2,546 (42.5%) returned no usable price data** from `yfinance`. This is a meaningful limitation, not a routine data gap: securities that were delisted, went bankrupt, or were acquired under financial distress are simultaneously more likely to fail a price download *and* more likely to represent the severe-loss cases this study's label is designed to detect. This exclusion may therefore not be random with respect to outcome, and could bias the resulting model toward underestimating concentration risk. This should be treated as an explicit limitation when interpreting model results, not as missing data that can be safely ignored.
- **No filing-date field:** the raw dataset provides only the reporting period (quarter-end date), not the actual date each 13F was filed. Since institutions have up to 45 days after quarter-end to file, using the quarter-end date directly as the point of "known information" risks a degree of look-ahead bias. In the absence of a true filing date in this dataset, the SEC's 45-day filing deadline will be used as a conservative proxy for the earliest point this information could plausibly have been acted upon, during label construction in Feature Engineering. This is documented here as an assumption to be applied at that stage, not as logic already implemented in this preprocessing pipeline.
- **Industry sector not yet sourced:** this pipeline resolves each holding to a *security type* (used to filter to equity instruments), not an *industry sector*. Industry sector, required for this study's sector-concentration feature, must be sourced separately in Feature Engineering.

## Assumptions

- Duplicate `(cusip, cik)` rows with genuinely differing values represent multiple sub-account or managed-portfolio positions under one parent institution, correctly combined by summing, rather than erroneous conflicting filings. This is considered the more likely interpretation given that such groups frequently contained several distinct, plausible non-zero values (in one case, six different values) rather than a single repeated figure, but is not certain.
- SEC Form 13F institutional holdings data reasonably approximates portfolio concentration behaviour relevant to STADIOEquities' retail investor risk problem, despite differences in investor type and scale (as previously documented in the project's RAAIDD log).