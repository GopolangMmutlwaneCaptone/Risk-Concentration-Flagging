# STADIOEQUITIES: Client Portfolio Risk Concentration Flagging

> A data science project to help STADIOEquities identify clients at risk of dangerous portfolio concentration before it results in financial loss, using an engineered concentration metric and supervised classification.

**Client:** STADIOEquities (digital investment platform)
**Author:** Gopolang Mmutlwane

## Table of Contents
- [Motivation](#motivation)
- [Problem Statement](#problem-statement)
- [Repository Structure](#repository-structure)
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

## Repository Structure

This repository is organised as follows:

| Folder | Purpose |
|---|---|
| `data/` | Raw and processed datasets used in this project |
| `notebooks/` | Exploratory data analysis notebooks |
| `src/preprocessing/` | Data cleaning and preprocessing scripts |
| `src/features/` | Feature engineering scripts |
| `src/models/` | Model training and inference code |
| `src/evaluation/` | Statistical helper and model comparison scripts |
| `src/visualisation/` | Charting and visualisation scripts |
| `artifacts/` | Saved trained model artifacts |
| `experiments/setup/` | Experimental setup (configs, train/test split logic, etc.) |
| `experiments/results/` | Experimental results (metrics, output tables, saved reports) |
| `reports/` | Client-facing written reports and recommendations |
| `requests/` | Client-facing data requests (data request PDF) |
| `literature-review/` | SS2 literature review and public dataset description |

Each folder contains its own `README.md` describing its contents in more detail.

---

## Data Request

The data requested from STADIOEquities to carry out this project is detailed in [`requests/data_request.pdf`](requesSts/data_request.pdf).

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