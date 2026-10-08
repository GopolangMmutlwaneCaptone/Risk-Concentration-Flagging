# SS3 Part B

Applies the model chosen in SS2 (XGBoost) to the client data extract supplied by STADIOEquities for the capstone, from dataset loading through to results visualisation. The notebook reads only the client extract and does not depend on any SS2 output. For the results, see [Results at a Glance](../README.md#results-at-a-glance) in the root README.

## Running the notebook

1. Set up the environment from the root [`requirements.txt`](../requirements.txt), which was built on Python 3.14 (see [Setup and Running the Notebooks](../README.md#setup-and-running-the-notebooks)).
2. Place the four `table*.csv` files of the client extract in `data/raw/client_extract/`. The extract is git-ignored and not included in this repository; see the [extract README](../data/raw/client_extract/README.md) for the tables.
3. Run `SS3_PartB.ipynb` top to bottom with `SS3_PartB/` as the working directory, since the notebook reads from `../data/raw/client_extract` and saves its charts to `charts/`.

## Contents

- `SS3_PartB.ipynb`: the notebook
- `requirements.txt`: the packages the notebook uses, with the same pinned versions as the root `requirements.txt`
- `charts/`: the charts the notebook saves
  - `missing_values.png`
  - `raw_categorical_fragmentation.png`
  - `client_age_outlier.png`
  - `cleaning_before_after.png`
  - `feature_distributions.png`
  - `feature_correlation.png`
  - `risk_appetite_distribution.png`
  - `hhi_by_risk_appetite.png`
  - `results_evaluation.png`
