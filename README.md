# stadioequities: Client Portfolio Risk Concentration Flagging

> A data science project to help STADIOEquities identify clients at risk of dangerous portfolio concentration before it results in financial loss, using an engineered concentration metric and supervised classification.

**Client:** STADIOEquities (digital investment platform)
**Author:** Gopolang Mmutlwane

## Table of Contents
- [Motivation](#motivation)
- [Problem Statement](#problem-statement)
- [Repository Structure](#repository-structure)
- [Data Request](#data-request)
- [RAAIDD Log](#raaidd-log)
- [Related Documentation](#related-documentation)

## Motivation
STADIOEquities is a digital investment platform operating in South Africa's retail investing and fintech sector. The platform has lowered the barrier to stock market participation dramatically, enabling first-time investors to open an account and buy fractional shares from their phones within minutes. This accessibility has attracted a large, young client base of approximately 2.3 million registered accounts, with a median client age of 31. Unlike traditional stockbroking models that catered to wealthier, more experienced investors, STADIOEquities' self-directed model places investment decisions entirely in the hands of often inexperienced, first-time clients.

Because STADIOEquities cannot yet distinguish between different types of investors on its platform, all clients receive the same content, nudges, and product exposure regardless of their behaviour or risk profile. This lack of differentiation has a tangible consequence: some inexperienced clients concentrate their entire account balance in a single volatile stock, exposing themselves to significant, avoidable risk. Under the current operating model, this concentration typically goes unnoticed until it results in a loss, at which point the client's dissatisfaction surfaces as a complaint. Complaints relating to unsuitable product or investment choices have been reported as a rising concern, yet no proactive mechanism currently exists to detect concentrated, high-risk positions before losses occur.

This issue matters on two levels. For individual clients, particularly first-time investors with limited market experience, concentrated exposure to a single volatile stock can result in disproportionate financial losses that undermine long-term trust in investing and in the platform itself. For STADIOEquities, unresolved concentration risk translates directly into rising complaint volumes, reputational exposure, and weakened client retention, all of which work against the platform's broader strategic goal of converting first-time sign-ups into long-term, engaged investors. Addressing this issue proactively, rather than reactively through complaint handling, aligns directly with STADIOEquities' stated priority of protecting clients from unsuitable investment behaviour before harm occurs.

Data science offers a practical means of identifying concentration risk earlier than current operations allow. By analysing holdings data, it is possible to quantify each portfolio's concentration using a measure such as the Herfindahl-Hirschman Index, and to combine this with other portfolio characteristics, such as top-holding percentage, number of distinct holdings, and sector exposure, to predict whether a portfolio is at elevated risk of a concentration-amplified loss in the following period, relative to a market benchmark. Rather than relying on complaints as the primary signal that something has gone wrong, such an approach could support STADIOEquities' client service and product teams in intervening earlier, for example, through timely guidance or diversification prompts before a loss occurs. This would not eliminate investment risk, which is inherent to any market participation, but it may meaningfully assist STADIOEquities in identifying and supporting clients who are exposed to avoidable, concentration-driven risk.

---

## Problem Statement
Despite STADIOEquities' current approach of identifying unsuitable investment behaviour only after a client has already suffered a loss and lodged a complaint, it remains unclear whether portfolio concentration measures, such as the Herfindahl-Hirschman Index, top-holding percentage, number of distinct holdings, and sector exposure, can be used to predict which portfolios are at elevated risk of a concentration-amplified loss relative to a market benchmark. Therefore, this study aims to build and evaluate a predictive model using SEC Form 13F institutional holdings data, a publicly available proxy for client portfolio behaviour, in order to support earlier, proactive intervention by STADIOEquities before losses occur.

---

## Repository Structure

This repository is organised as follows:

| Folder | Purpose |
|---|---|
| `data/` | Raw and processed datasets used in this project |
| `notebooks/` | Exploratory and preprocessing notebooks |
| `src/models/` | Model training and inference code |
| `src/evaluation/` | Statistical helper and model comparison scripts |
| `src/visualisation/` | Charting and visualisation scripts |
| `experiments/setup/` | Experimental setup (configs, train/test split logic, etc.) |
| `experiments/results/` | Experimental results (metrics, output tables, saved reports) |
| `requests/` | Client-facing deliverables (e.g. data request PDF) |

Each folder contains its own `README.md` describing its contents in more detail.

---

## Data Request

The data requested from STADIOEquities to carry out this project is detailed in [`requests/data_request.pdf`](requests/data_request.pdf).

---

## RAAIDD Log

The project's Risks, Actions, Assumptions, Issues, Decisions, and Dependencies are documented in [`requests/raaidd_log.pdf`](requests/raaidd_log.pdf).

---

## Related Documentation

<!-- Placeholder section. will link out to Preprocessing.MD, FeatureEngineering.MD, 
     Model1.MD, Model2.MD, etc. once SS2 begins -->
=======
