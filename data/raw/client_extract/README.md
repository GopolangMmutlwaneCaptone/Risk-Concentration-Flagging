# STADIOEquities — Client Concentration Risk: Data Extract

The client-level dataset described in the data request, for direct application of the concentration-risk methodology to STADIOEquities' own client base. `client_id` is the common join key across all four tables and contains no personal information. Dates are `yyyy-MM-dd`; the transaction timestamp is `yyyy-MM-dd HH:mm:ss`. All values are ZAR.

## Files

### `table1_client_account_information.csv`
One row per client — demographics, stated risk profile and account status.  
Rows: 419

| Column | Description |
|---|---|
| `client_id` | String |
| `client_age` | Integer |
| `account_open_date` | Date |
| `client_province` | Categorical |
| `stated_risk_appetite` | Categorical |
| `stated_investment_goal` | Categorical |
| `account_status` | Categorical |

### `table2_portfolio_holdings.csv`
Position-level holdings at monthly snapshots.  
Rows: 45,660

| Column | Description |
|---|---|
| `client_id` | String |
| `snapshot_date` | Date |
| `instrument_ticker` | String |
| `exchange` | Categorical |
| `instrument_name` | String |
| `sector` | Categorical |
| `quantity_held` | Numeric |
| `market_value_of_holding_zar` | Numeric |
| `total_portfolio_value_zar` | Numeric |

### `table3_transaction_history.csv`
One row per executed buy or sell.  
Rows: 4,175

| Column | Description |
|---|---|
| `client_id` | String |
| `transaction_id` | String |
| `transaction_date` | Datetime |
| `transaction_type` | Categorical |
| `order_type` | Categorical |
| `instrument_ticker` | String |
| `exchange` | Categorical |
| `quantity` | Numeric |
| `price_per_unit_zar` | Numeric |
| `transaction_value_zar` | Numeric |

### `table4_support_complaint_history.csv`
Complaints and support tickets, raw free text.  
Rows: 130

| Column | Description |
|---|---|
| `client_id` | String |
| `complaint_id` | String |
| `date_logged` | Date |
| `complaint_text` | Categorical |
| `complaint_category` | Categorical |
| `related_instrument_ticker` | String |
| `resolution_outcome` | Categorical |

## Data quality notes

This is an extract from live operational systems and has **not** been cleaned. Expect:

- Missing values in optional / conditionally-populated fields
- Inconsistent casing and stray whitespace in some free-text and categorical fields
- A small number of duplicate records
- Occasional out-of-range or implausible values from data-capture errors
- Some date fields captured in inconsistent formats

**Snapshot cadence: monthly, as strongly preferred.** `table2_portfolio_holdings.csv` holds **12 monthly month-end snapshots** to 31 August 2026, not quarterly. Aggregate to quarter-end if you need consistency with the 13F proxy dataset.

**`total_portfolio_value_zar` is deliberately repeated** on every holding row for a given client-date, exactly as specified — it is the sum of equity holdings' market value only, excluding uninvested cash, for consistency with Form 13F. Holding weight = `market_value_of_holding_zar / total_portfolio_value_zar`. This is not a normalisation error.

**`order_type` is supplied.** STADIOEquities does offer a recurring/auto-invest feature, so the column is populated rather than omitted: `Manual` or `Recurring/Auto-invest`. Roughly a third of clients use auto-invest, which is what lets you separate deliberate concentration from passive accumulation.

**`complaint_category` is only partially populated — this is real.** Support queries are logged as unstructured free text and only about 38% have ever been manually categorised. The blanks are genuine absence of categorisation, not missing data, so `complaint_text` is the primary field and deriving categories from it is part of the work. `related_instrument_ticker` is likewise blank wherever the client did not name an instrument.

**Signals present:** clients whose observed concentration contradicts their stated risk appetite (low stated risk, high observed concentration) are over-represented among suitability complaints. That mismatch is the intended outcome label for validating the concentration indicators.

**Fractional quantities** are expected throughout — `quantity_held` and `quantity` are floats to 4 decimal places, since fractional share ownership is supported.

_Synthetic data supplied for the CAP182 capstone project. Not real client data._