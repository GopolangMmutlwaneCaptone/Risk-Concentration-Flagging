# SS3: Applying the Approach to Client Data

SS3 is the final stage of the capstone for the client STADIOEquities. It applies the approach validated in SS2 (XGBoost, chosen on the public SEC Form 13F proxy dataset) to the client data extract supplied by STADIOEquities, and it contains the written peer review and the recorded presentation.

## Contents

| Folder | What it contains |
|---|---|
| [`PartA/`](PartA/) | Part A: the written peer review of a classmate's capstone project (PDF) |
| [`PartB/`](PartB/) | Part B: the notebook that retrains XGBoost on the client extract, with its charts and requirements |
| [`video_Presentation/`](video_Presentation/) | The recorded video presentation of the whole project |

## Part B at a glance

- **Data:** 418 clients from the STADIOEquities extract (synthetic data), of whom 17 (4.07%) are flagged. A client is flagged when stated risk appetite is Low and portfolio concentration (HHI) is above the 75th percentile.
- **Split:** stratified 70/15/15, giving 292, 63 and 63 clients with 12, 3 and 2 flagged clients.
- **Results:** ROC-AUC 0.856 on validation (1 of 3 flagged clients caught) and 0.951 on the unseen test set (2 of 2 caught).
- **Main finding:** a simple rule that flags every client above the HHI threshold performs as well as the model on this label (cross-validated ROC-AUC 0.895 for HHI alone against 0.887 for the model), so the rule is the better first-pass screen. The flag could not be validated against complaint records, so it is a business-defined rule.
- **Caution:** with only 3 and 2 flagged clients in the validation and test sets, these figures are early signals, not stable estimates.

For the full set of results and known limitations, see [Results at a Glance](../README.md#results-at-a-glance) and [Known Limitations](../README.md#known-limitations) in the root README.

## Running Part B

See [`PartB/README.md`](PartB/README.md). The client extract is not included in this repository.

## Navigation

- [Back to the root README](../README.md)
