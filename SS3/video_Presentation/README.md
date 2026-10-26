# Presentation Video

This folder contains the recorded video presentation of the capstone project for STADIOEquities, together with the slide deck used in the video.

## Contents

- [`Video_Presentation_Risk_Concentration_Flagging.mp4`](Video_Presentation_Risk_Concentration_Flagging.mp4): the presentation video
- [`Risk_Concentration_Flagging_Capstone_Presentation.pptx`](Risk_Concentration_Flagging_Capstone_Presentation.pptx): the PowerPoint slide deck shown in the video (13 slides)

## What the video covers

1. The real-world problem: clients concentrating their money in a single stock, and complaints surfacing only after a loss.
2. How the project was tackled: framing the problem, testing the approach on public data, and applying it to client data.
3. The two datasets: public SEC Form 13F filings (used as a stand-in while client data was not yet available) and the STADIOEquities client extract.
4. Data preparation, and the maths behind the concentration measure (HHI) and the flag.
5. Exploratory insights from the client data.
6. The method: Logistic Regression compared with XGBoost.
7. Results on the public data, then results on the client data.
8. Recommendations and limitations.

## Where to find the full results

The slides summarise the findings. For the full results and known limitations, see [Results at a Glance](../../README.md#results-at-a-glance) and [Known Limitations](../../README.md#known-limitations) in the root README, and the Part B notebook in [`../PartB/`](../PartB/).

[Back to SS3 overview](../README.md)