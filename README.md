# Detecting Authorized Push Payment Fraud in UPI

**A Behavioral Feature Engineering Approach Using PaySim**

Solo research paper — B.Tech CSE (Fintech Honours), 3rd Year.

---

## Abstract

India's Unified Payments Interface (UPI) is dominated by **Authorized Push Payment (APP)
fraud**, in which the genuine account holder is socially engineered into initiating the
transfer themselves — fake collect requests, swapped merchant QR codes, screen-sharing
"support" scams. FY24 recorded 13.42 lakh UPI fraud cases worth roughly ₹1,087 crore.

This breaks the assumption behind most fraud-detection literature, which is built around
account-takeover-style anomaly detection where the transacting party is *not* the account
owner. An APP-fraud transaction is authorized by the real holder, from their own device,
with their own PIN — so it carries no anomaly signature at the account level.

Phase 1 established the gap and a behavioral-feature baseline on PaySim-style data.
Phase 2 moves the detection signal off the account and onto the **payment graph and its
temporal dynamics**, in the **cross-institutional** setting UPI actually operates in.

## Status

| Phase | Scope | State |
|---|---|---|
| Phase 1 | Literature review, behavioral feature engineering, Logistic Regression + Decision Tree baseline on a schema-faithful PaySim proxy | Submitted — `Phase1_UPI_Fraud_Detection.docx` |
| Phase 2 | Research for development, building on references **[3] HiFraud** and **[4] Heterophily-aware temporal GNN** | In progress — see [`docs/phase2_research_plan.md`](docs/phase2_research_plan.md) |

## Setup

The project virtual environment lives **outside this repo** at `D:\temp\new env`
(Python 3.10 — chosen over 3.14 for ML wheel coverage).

```powershell
# Create (already done)
py -3.10 -m venv "D:\temp\new env"

# Install
& "D:\temp\new env\Scripts\python.exe" -m pip install -r requirements.txt

# Run anything with the venv interpreter directly — note the space in the path
& "D:\temp\new env\Scripts\python.exe" src\train.py
```

> The path contains a space. Quote it in every command.

## Data

No dataset is committed — raw data is gitignored and not redistributable.

| Dataset | Use | Source |
|---|---|---|
| PaySim | Phase 1 baseline, Phase 2 tabular reference | [kaggle.com/datasets/ealaxi/paysim1](https://www.kaggle.com/datasets/ealaxi/paysim1) |
| MoMTSim | Phase 2 primary — richer fraud scenarios | Reference [1], ScienceDirect 2025 |

Download into `data/raw/`.

**Scope honesty:** genuine UPI transaction records are not released publicly by NPCI.
Every experiment here runs on mobile-money *proxies*. Claims are scoped to what those
proxies can support — see the limitations section of the Phase 2 plan.

## Layout

```
data/raw/          # downloaded datasets (gitignored)
data/processed/    # feature-engineered outputs (gitignored)
notebooks/         # exploration
src/               # pipeline code
results/           # metrics, figures
docs/              # phase2_research_plan.md
references/        # reference papers
```
