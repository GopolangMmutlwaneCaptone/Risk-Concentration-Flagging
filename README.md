# STADIOEQUITIES: Client Portfolio Risk Concentration Flagging

> A data science project to help STADIOEquities identify clients at risk of dangerous portfolio concentration before it results in financial loss, using an engineered concentration metric and supervised classification.

**Client:** STADIOEquities (digital investment platform)
**Author:** Gopolang Mmutlwane

## Table of Contents
- [Motivation](#motivation)
- [Problem Statement](#problem-statement)
- [Project Stages](#project-stages)
- [Results at a Glance](#results-at-a-glance)
- [Repository Structure](#repository-structure)
- [Dataset](#dataset)
- [Setup and Running the Notebooks](#setup-and-running-the-notebooks)
- [Reproducibility](#reproducibility)
- [Data Request](#data-request)
- [RAAIDD Log](#raaidd-log)
- [Literature Review](#literature-review)
- [Related Documentation](#related-documentation)

## Motivation
STADIOEquities is a digital investment platform operating in South Africa's retail investing and fintech sector. The platform has lowered the barrier to stock market participation dramatically, enabling first-time investors to open an account and buy fractional shares from their phones within minutes. This accessibility has attracted a large, young client base of approximately 2.3 million registered accounts, with a median client age of 31. Unlike traditional stockbroking models that catered to wealthier, more experienced investors, STADIOEquities' self-directed model places investment decisions entirely in the hands of often inexperienced, first-time clients.

Because STADIOEquities cannot yet distinguish between different types of investors on its platform, all clients receive the same content, nudges, and product exposure regardless of their behaviour or risk profile. This lack of differentiation has a tangible consequence: some inexperienced clients concentrate their entire account balance in a single volatile stock, exposing themselves to significant, avoidable risk. Under the current operating model, this concentration typically goes unnoticed until it results in a loss, at which point the client's dissatisfaction surfaces as a complaint. Complaints relating to unsuitable product or investment choices have been reported as a rising concern, yet no proactive mechanism currently exists to detect concentrated, high-risk positions before losses occur.

This issue matters on two levels. For individual clients, particularly first-time investors with limited market experience, concentrated exposure to a single volatile stock can result in disproportionate financial losses that undermine long-term trust in investing and in the platform itself. For STADIOEquities, unresolved concentration risk translates directly into rising complaint volumes, reputational exposure, and weakened client retention, all of which work against the platform's broader strategic goal of converting first-time sign-ups into long-term, engaged investors. Addressing this issue proactively, rather than reactively through complaint handling, aligns directly with STADIOEquities' stated priority of protecting clients from unsuitable investment behaviour before harm occurs.

Data science offers a practical means of identifying concentration risk earlier than current operations allow. By analysing holdings data, it is possible to quantify each portfolio's concentration using a measure such as the Herfindahl-Hirschman Index, and to combine this with other portfolio characteristics, such as top-holding percentage, number of distinct holdings, and sector exposure, to predict whether a portfolio is at elevated risk of a concentration-amplified loss in the following period, relative to a market benchmark. Rather than relying on complaints as the primary signal that something has gone wrong, such an approach could support STADIOEquities' client service and product teams in intervening earlier, for example, through timely guidance or diversification prompts before a loss occurs. This would not eliminate investment risk, which is inherent to any market participation, but it may meaningfully assist STADIOEquities in identifying and supporting clients who are exposed to avoidable, concentration-driven risk.

---

## Problem Statement

Despite STADIOEquities' current approach of identifying unsuitable investment behaviour only after a client has already suffered a loss and lodged a complaint, it remains unclear how portfolio concentration measures, such as the Herfindahl-Hirschman Index, top-holding percentage, number of distinct holdings, and sector exposure, can best be used within a predictive model to identify which portfolios are at elevated risk of a concentration-amplified loss relative to a market benchmark. Therefore, this study aims to build and evaluate a predictive model using client portfolio and transaction data requested from STADIOEquities, in order to support earlier, proactive intervention before losses occur.

---

## Project Stages

The project was delivered in three stages. Each stage builds on the previous one.

| Stage | What it covers | Where to look |
|---|---|---|
| **SS1** | Problem selection and framing, the data request to STADIOEquities, and the RAAIDD log | [Problem Statement](#problem-statement), [Data Request](#data-request), [RAAIDD Log](#raaidd-log) |
| **SS2** | The methodology, validated on a public proxy dataset (SEC Form 13F institutional holdings): literature review, preprocessing, feature engineering, label construction, two models (logistic regression and XGBoost), a statistical comparison, and a recommendations report | [Literature Review](#literature-review), [Related Documentation](#related-documentation), [Recommendations Report](reports/recommendations_report.pdf) |
| **SS3** | Part B: the model chosen in SS2 (XGBoost) applied to STADIOEquities' real client data extract, from dataset loading through to results visualisation | [`SS3_PartB/SS3_PartB.ipynb`](SS3_PartB/SS3_PartB.ipynb) |

---

## Results at a Glance

**SS2: public proxy dataset.** 20,244 portfolio-period rows (846 positive), evaluated on a held-out test set of 5,093 rows (114 positive).

| Metric (positive class) | Model 1: Logistic Regression | Model 2: XGBoost |
|---|---|---|
| ROC-AUC | 0.921 | 0.926 |
| Recall | 0.78 | 0.94 |
| Precision | 0.13 | 0.09 |

XGBoost was recommended because catching at-risk portfolios matters more here than avoiding false alarms. A DeLong test found no significant difference in ROC-AUC (p = 0.210), while McNemar's test showed the two models classify different cases correctly. Full detail is in [Model Comparison](experiments/results/Comparison.MD).

**SS3: STADIOEquities client extract.** 418 clients, of whom 17 (4.07%) are flagged by the label (stated risk appetite is Low and portfolio concentration is above the 75th percentile). The split is stratified 70/15/15, giving 12, 3 and 2 positive clients in the training, validation and test sets.

| Set | ROC-AUC | Positives caught | Precision (positive class) |
|---|---|---|---|
| Validation (63 clients) | 0.856 | 1 of 3 | 0.17 |
| Test, unseen (63 clients) | 0.951 | 2 of 2 | 0.29 |

Points to read alongside these numbers:
- With only 2 to 3 positive clients per evaluation set, each metric is decided by one or two individual clients and should be read as an early signal, not a stable estimate.
- In validation, the model missed the single most concentrated client (one holding, concentration index 1.0) because nothing like it appeared among the 12 training positives. The notebook recommends running a simple absolute rule for extreme concentration alongside the model.
- The label is a business-defined construct. It could not be validated against suitability complaints in this extract (Mann-Whitney U tests, p = 0.738 for the concentration index and p = 0.840 for top-holding percentage), and the notebook reports this as a finding rather than omitting it.
- The extract is synthetic data supplied for the module.

---

## Repository Structure

This repository is organised as follows:

| Folder | Purpose |
|---|---|
| `data/` | Raw and processed datasets used in this project. Raw data is not committed here (see [Dataset](#dataset) below). `data/raw/client_extract/` is where the STADIOEquities client extract for SS3 is placed; only its README is committed. |
| `src/preprocessing/` | SS2 data cleaning and preprocessing notebook and documentation |
| `src/features/` | SS2 feature engineering notebook and documentation |
| `src/models/` | SS2 model notebooks (logistic regression, XGBoost) and documentation |
| `SS3_PartB/` | SS3 Part B notebook applying the chosen model to the client extract, and the charts it produces |
| `artifacts/` | Saved trained model artifacts, generated locally by running the model notebooks (not committed) |
| `experiments/results/` | Experimental results (metrics, comparison, performance documents) |
| `reports/` | Client-facing written reports and recommendations, with charts in `reports/charts/` |
| `requests/` | Client-facing data requests (data request PDF) |
| `literature-review/` | SS2 literature review and public dataset description |
| `notebooks/`, `src/evaluation/`, `src/visualisation/`, `experiments/setup/` | Placeholder folders that currently contain only a README; the analysis itself lives in the notebooks under `src/` |

Most folders contain their own `README.md` describing their contents in more detail.

---

## Dataset

**SS2 (public proxy dataset).** The raw SEC Form 13F dataset is not committed to this repository due to its size (over 300MB). Before running `preprocessing.ipynb`, download it from [Kaggle](https://www.kaggle.com/datasets/aneeshpanoli/sec-13fhr-institutional-investment-data) and place `13Fdata.csv`, `institutions.csv`, and `stock_names.csv` in `data/raw/`. All other data files (mapped tickers, price history, engineered features) are generated automatically by running the notebooks in order.

**SS3 (STADIOEquities client extract).** The extract consists of four CSV tables: client account information, portfolio holdings, transaction history, and support complaint history. It is client data and is not distributed with this repository. To run the SS3 notebook, place the four `table*.csv` files in `data/raw/client_extract/`; the [extract README](data/raw/client_extract/README.md) describes each table and its known data quality issues.

---

## Setup and Running the Notebooks

The pinned environment in `requirements.txt` was built on Python 3.14.

```powershell
# Windows (PowerShell)
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

```bash
# macOS / Linux
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Then open the notebooks in VS Code or Jupyter and select the `.venv` interpreter as the kernel (`ipykernel` is included in `requirements.txt`). Run each notebook top to bottom with a fresh kernel.

**SS2 run order** (each step reads the files written by the one before it):

1. `src/preprocessing/preprocessing.ipynb`: ticker mapping and price data. This is the slow step; the ticker lookup and sector lookup can take several hours in total, and intermediate results are checkpointed to `data/processed/`.
2. `src/features/feature_engineering.ipynb`
3. `src/models/model1_logistic_regression.ipynb`
4. `src/models/model2_xgboost.ipynb`, which must run after Model 1 because it loads the model and scaler Model 1 saves to `artifacts/`.

**SS3:** after placing the client extract (see [Dataset](#dataset)), run `SS3_PartB/SS3_PartB.ipynb`. It reads only the client extract and does not depend on any SS2 output.

---

## Reproducibility

- **Seeds.** Every model, data split and cross-validation step sets a fixed random seed, and results are deterministic within a given environment. The SS3 notebook uses `random_state=42` throughout.
- **SS2 environment.** The committed SS2 results were produced on Python 3.11 with the package versions recorded in `src/models/requirements.txt` (including xgboost 2.0.3).
- **Re-running on a newer environment.** On Python 3.14 with xgboost 3.4.1, Model 1 reproduces exactly, while Model 2 shifts slightly: ROC-AUC 0.924 instead of 0.926, and the DeLong p-value 0.41 instead of 0.21. Recall (0.94) is unchanged, and the conclusions are the same. The documents in this repository quote the committed values.
- **SS3 environment.** The SS3 notebook was run on the environment pinned in `requirements.txt`.

---

## Data Request

The data requested from STADIOEquities to carry out this project is detailed in [`requests/data_request.pdf`](requests/data_request.pdf).

---

## RAAIDD Log

| Category | Description |
|---|---|
| **Risk** | Concentration flagging alone identifies at-risk clients but does not address the underlying reasons clients concentrate their holdings (e.g. chasing trending stocks, lack of investing knowledge); without pairing the model with client education or guidance, STADIOEquities may see limited real-world impact from flags alone. |
| **Risk** | The proxy dataset used to validate this methodology (institutional holdings) differs meaningfully from STADIOEquities' actual retail client base, which may limit how well findings generalise once applied to real client data. |
| **Risk** | If STADIOEquities does not have structured historical records of past client complaints, constructing a real-world validation label in a future direct application may be more difficult than anticipated. |
| **Risk** | Client-level financial and behavioural data requested from STADIOEquities is sensitive under South Africa's POPIA regulations; any future direct application must ensure proper anonymisation and lawful basis for processing. |
| **Action** | Explicitly document the proxy-to-real-client limitation in the final report, so STADIOEquities understands the current study validates a methodology rather than directly modelling its own clients. |
| **Action** | Engage STADIOEquities stakeholders (e.g. client service and product teams) early in any future direct-application phase, to ensure flagged clients are met with appropriate, supportive interventions rather than purely automated action. |
| **Action** | Maintain incremental, clearly-described Git commits throughout development, in line with the module's emphasis on commit history as a reviewable record of progress. |
| **Action** | Empirically check the class balance of the constructed risk label before committing to accuracy as an evaluation metric, since concentration-driven losses are likely to be a minority outcome. |
| **Assumption** | STADIOEquities' stated strategic priority of "protecting clients from unsuitable investment behaviour" reflects a genuine willingness to act on model outputs (e.g. through guidance or diversification prompts), not just awareness of the risk. |
| **Assumption** | The proxy dataset reasonably approximates portfolio concentration behaviour relevant to STADIOEquities' retail investor risk problem, despite differences in investor type and scale. |
| **Assumption** | STADIOEquities would, in a real engagement, be able to provide the client-level data described in the Part C data request, including anonymised holdings and transaction history. |
| **Assumption** | Market benchmark data needed to construct the risk label is freely available and can be joined to the proxy dataset without licensing restrictions. |
| **Issue** | Early in repository setup, a Git authentication mismatch (local credentials cached for a different GitHub account than the one hosting the repository) caused repeated push failures until diagnosed and resolved. |
| **Issue** | A `.gitignore` exclusion rule was silently corrupted due to a text-encoding issue, causing it to fail without any visible error until identified through direct inspection. |
| **Decision** | Risk Concentration Flagging was selected as the project's problem statement over six other candidate problems, based on alignment with STADIOEquities' stated strategic priorities, available public data, and literature support. |
| **Decision** | The prediction target was defined at the portfolio level (concentration-weighted excess return), rather than based on a single holding in isolation, to ensure the model's outcome is mechanically tied to concentration itself, matching the real risk STADIOEquities is trying to address. |
| **Dependency** | The future direct-application phase depends on STADIOEquities agreeing to and providing the data described in the Part C request. |
| **Dependency** | Any client-facing intervention (e.g. guidance or diversification prompts) depends on STADIOEquities' product and client service teams building the operational workflow to act on model outputs. |
| **Dependency** | The SS2 model comparison and recommendations report depend on the completion of preprocessing, feature engineering, and label construction using the proxy dataset. |
| **Dependency** | Validating this methodology against real STADIOEquities outcomes (e.g. actual complaints or losses) depends on STADIOEquities maintaining structured, joinable records of these events. |

---

## Literature Review

A review of three related publications and a description of the publicly available dataset used to validate this study's methodology is provided in [`literature-review/Related_work_and_data_description.pdf`](literature-review/Related_work_and_data_description.pdf).

---

## Related Documentation

- [Preprocessing](src/preprocessing/Preprocessing.MD): data cleaning, CUSIP-to-ticker mapping, and price data preparation for the SS2 public proxy dataset
- [Feature Engineering](src/features/FeatureEngineering.MD): portfolio concentration features and risk label construction
- [Model 1: Logistic Regression](src/models/Model1_Logistic_Regression.MD): interpretable baseline model, technique, and hyperparameters
- [Model 1 Performance](experiments/results/Model1_Logistic_Regression_Performance.MD): Model 1 test-set metrics and results
- [Model 2: XGBoost](src/models/Model2_XGBoost.MD): gradient-boosted ensemble model, technique, and hyperparameters
- [Model 2 Performance](experiments/results/Model2_XGBoost_Performance.MD): Model 2 test-set metrics and results
- [Model Comparison](experiments/results/Comparison.MD): side-by-side comparison and model recommendation
- [Recommendations Report](reports/recommendations_report.pdf): model recommendations, model improvement suggestions, and alignment with literature
- [SS3 Part B notebook](SS3_PartB/SS3_PartB.ipynb): the chosen model applied to the STADIOEquities client extract
