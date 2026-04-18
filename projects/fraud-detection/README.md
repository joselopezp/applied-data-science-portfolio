# Fraud Detection System: Cost-Optimized Classification for Financial Transactions

> **CRISP-DM + Lean — PBL Project** | applied-data-science-portfolio

![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![Framework](https://img.shields.io/badge/Framework-CRISP--DM%20%2B%20LEAN-2E86AB)
![Type](https://img.shields.io/badge/Type-Supervised%20ML-blueviolet)
![License](https://img.shields.io/badge/License-MIT-yellow)

---

## Executive Summary

Financial institutions lose billions annually to fraud — but the real problem is asymmetric:
missing fraud costs 20x more than blocking a legitimate transaction. This project reframes
fraud detection as a **cost minimization problem**, not a classification problem. A Random
Forest model calibrated at threshold t*=0.0414 catches 81.7% of fraud with an Expected Loss
of USD 4,320 — a **79.7% reduction** vs the naive baseline of USD 21,300.

---

## Business Objective

**Problem:** Fraud detection systems fail in two directions simultaneously — undetected fraud
(direct loss) and blocked legitimate transactions (customer friction). Standard accuracy
metrics hide both failures: a model that labels everything as legitimate achieves 99.8%
accuracy while catching zero fraud.

**Why it matters:** Global card fraud losses exceed USD 30B annually. A miscalibrated model
exposes the institution to the full cost of undetected fraud.

**Decision to support:** At what classification threshold should a transaction be flagged as
fraudulent, given a defined cost ratio between False Negatives and False Positives?

**Expected impact:** 79.7% reduction in Expected Loss vs naive baseline — USD 16,980 saved
per test window of 42,721 transactions.

---

## Problem Statement Canvas

| Element | Content |
|---|---|
| **Business Problem** | Fraud detection systems face two simultaneous failure modes: undetected fraud (direct loss) and blocked legitimate transactions (customer friction). Accuracy metrics hide both. |
| **Business Impact** | Global card fraud losses exceed USD 30B annually. Naive classifier achieves 99.83% accuracy while catching zero fraud. |
| **Decision to Support** | At what threshold should a transaction be flagged as fraud, given asymmetric costs: cost_fn=$300 vs cost_fp=$15 (ratio 20x)? |
| **Analytical Question** | Can a supervised classifier with cost-calibrated threshold achieve Recall ≥ 0.80 and AUC-PR ≥ 0.85 on the fraud class? |
| **Success Metrics** | Recall ≥ 0.80 ✅ · Precision ≥ 0.70 ❌ · AUC-PR ≥ 0.85 ❌ · Expected Loss < naive baseline ✅ |
| **Proposed Approach** | Logistic Regression (baseline) → Random Forest → XGBoost × 3 imbalance strategies. Threshold calibrated by Expected Loss function. Isolation Forest as unsupervised benchmark. |

---

## Solution Overview

| Field | Details |
|---|---|
| **Model** | Random Forest Classifier + class_weight='balanced' |
| **Threshold** | t* = 0.0414 (cost-optimized via Expected Loss function) |
| **Cost ratio** | 20x — missing fraud costs 20x more than blocking a legitimate transaction |
| **Imbalance handling** | class_weight='balanced' — winner over SMOTE and no-balancing |
| **Primary metric** | AUC-PR (not AUC-ROC — misleading with 0.17% fraud rate) |
| **Dataset** | 284,807 credit card transactions · 492 fraud (0.17%) |
| **End users** | Risk/fraud teams · CFO · Customer experience teams |
| **Output** | Fraud flag + probability score + Expected Loss at configurable cost ratio |
| **Dependencies** | scikit-learn · imbalanced-learn · xgboost · joblib |

---

## Repository Structure

```
fraud-detection/
├── README.md
├── requirements.txt
├── data/
│   ├── raw/          ← original data (excluded via .gitignore)
│   ├── processed/    ← scaled splits: X_train, X_val, X_test, SMOTE variants
│   └── final/
├── notebooks/
│   ├── 01_business_understanding.ipynb   ← Problem canvas + Expected Loss Framework
│   ├── 02_data_understanding.ipynb       ← EDA + discriminative feature ranking
│   ├── 03_data_preparation.ipynb         ← Scaling + split + imbalance strategies
│   ├── 04_modeling.ipynb                 ← 9 configs + Isolation Forest benchmark
│   ├── 05_evaluation.ipynb               ← Threshold calibration + final test results
│   └── 06_deployment.ipynb               ← Model card + MLOps + executive summary
├── models/
│   └── model_v1_2026-04.pkl
├── reports/
│   ├── figures/                          ← generated plots (excluded via .gitignore)
│   ├── executive_summary_EN.md
│   └── executive_summary_ES.md
├── src/
│   └── __init__.py
└── docs/
    ├── decisions_log.md
    └── lean_retrospective.md
```

---

## CRISP-DM Phases

| Notebook | Phase | Key Output |
|---|---|---|
| 01 | Business Understanding | Problem Statement Canvas · Expected Loss Framework · Cost matrix |
| 02 | Data Understanding | Class imbalance analysis · Top discriminative PCA features · Data quality report |
| 03 | Data Preparation | StandardScaler on Amount/Time · Stratified 70/15/15 split · SMOTE vs class_weight comparison |
| 04 | Modeling | 9 model configurations compared · RF class_weight wins (AUC-PR val=0.8584) · Isolation Forest benchmark |
| 05 | Evaluation | t*=0.0414 calibrated · AUC-PR=0.7861 · Recall=0.8169 · EL=USD 4,320 · Savings=79.7% |
| 06 | Deployment | Model Card · MLOps checklist · Executive Summary · Streamlit app specification |

---

## Key Insights

| Insight | Context | Analysis | Possible Decision |
|---|---|---|---|
| Accuracy is the wrong metric | 0.17% fraud rate — naive model scores 99.83% accuracy catching zero fraud | Evaluate on AUC-PR and Expected Loss, never accuracy | Never report accuracy for imbalanced fraud detection |
| Default threshold (0.5) is suboptimal | Cost ratio 20x — FN is far more costly than FP | Optimal threshold shifts to t*=0.0414 — well below default | Always calibrate threshold by cost function in production |
| Labeled data adds significant value | Isolation Forest (unsupervised) achieves lower AUC-PR than all supervised models | Annotation cost of labeling fraud is justified by performance gain | Invest in maintaining labeled fraud datasets |
| class_weight beats SMOTE | RF + class_weight wins leaderboard over RF + SMOTE | Synthetic data does not improve over native weighting for this dataset | Prefer class_weight as first imbalance strategy |

---

## Model Card

| Field | Details |
|---|---|
| **Model type** | Random Forest Classifier |
| **Imbalance strategy** | class_weight='balanced' |
| **Task** | Binary classification — fraud (1) vs legitimate (0) |
| **Training data** | ULB Credit Card Fraud — 284,807 transactions (September 2013, European cardholders) |
| **Features used** | 30: Time, Amount (StandardScaler), V1–V28 (PCA anonymized) |
| **Target variable** | Class — 1 = Fraud, 0 = Legitimate |
| **Framework** | Scikit-learn 1.8.0 |
| **Training date** | April 2026 |
| **Artifact** | `models/model_v1_2026-04.pkl` |
| **Threshold** | t* = 0.0414 (cost-optimized) |

---

## Model Performance

| Metric | Validation | Test | Target | Met? |
|---|---|---|---|---|
| **AUC-PR** (primary) | 0.8584 | 0.7861 | ≥ 0.85 | ❌ |
| **Recall** (fraud) | — | 0.8169 | ≥ 0.80 | ✅ |
| **Precision** (fraud) | — | 0.6744 | ≥ 0.70 | ❌ |
| **F1** (fraud) | — | 0.7389 | — | — |
| **Expected Loss** | — | USD 4,320 | < naive | ✅ |
| **Savings vs naive** | — | USD 16,980 (79.7%) | > 0 | ✅ |

> **Baseline (naive — predict all legitimate):** USD 21,300 · 0% fraud caught
> **Model improvement:** 79.7% reduction in Expected Loss · 81.7% fraud caught

---

## MLOps Checklist

### Reproducibility
- [x] Random seed fixed (`random_state=42`)
- [x] Requirements pinned (`requirements.txt`)
- [x] Data versioned (Kaggle CLI command in notebook 02)
- [x] Model saved (`models/model_v1_2026-04.pkl`)

### Model Versioning
- [x] Artifact in `models/` — filename: `model_v1_2026-04.pkl`
- [x] Training parameters in Decisions Log (notebooks 03–05)
- [x] Model Card documented above

### Monitoring
- [x] Data drift noted — 2013 dataset, fraud patterns evolve over time
- [x] Limitations documented below
- [x] Retraining trigger defined — AUC-PR drop > 5%

---

## Business Recommendations

| Priority | Action | Expected Impact |
|---|---|---|
| 🔴 HIGH | Calibrate cost_fn and cost_fp with real institutional loss data | More accurate threshold — improves both precision and recall |
| 🔴 HIGH | Implement human review queue for all flagged transactions | Recovers false positives before customer impact — reduces friction |
| 🟡 MEDIUM | Retrain monthly with new transaction data | Prevents degradation as fraud patterns evolve |
| 🟡 MEDIUM | Test XGBoost with hyperparameter tuning | May close AUC-PR gap (0.7861 → 0.85 target) |
| 🟢 LOW | Wrap model in REST API for real-time scoring | Enables integration with transaction processing systems |

---

## Limitations

**Data:** Dataset covers 2 days in 2013 (European cardholders only). Features V1–V28 are
anonymized PCA components — no domain feature engineering possible. Cost assumptions
(cost_fn=$300, cost_fp=$15) are estimates, not real institutional loss data.

**Model:** AUC-PR target (0.85) not met — current model achieves 0.7861. Precision target
(0.70) not met — aggressive threshold t*=0.0414 produces more false positives as expected
under cost ratio 20x. Random Forest is not natively interpretable at transaction level.

**Deployment:** Model not tested on real-time streaming data. No automated retraining
pipeline. No A/B testing framework defined.

---

## How to Run

```powershell
# From portfolio root — activate shared virtual environment
.venv\Scripts\Activate.ps1

# Navigate to project
cd projects\fraud-detection

# Download dataset (required — excluded from git)
kaggle datasets download -d mlg-ulb/creditcardfraud -p data/raw/ --unzip

# Run notebooks in order
jupyter lab notebooks\01_business_understanding.ipynb
```

> ⚠️ Raw data excluded via `.gitignore`. Download required before running notebook 02.

---

## Data Source

| Field | Details |
|---|---|
| **Dataset** | [Credit Card Fraud Detection](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud) |
| **Provider** | ULB Machine Learning Group — Kaggle |
| **License** | DbCL v1.0 (Open Database License) |
| **Records** | 284,807 rows × 31 columns |
| **Accessed** | April 2026 |

```bash
kaggle datasets download -d mlg-ulb/creditcardfraud -p data/raw/ --unzip
```

---

## Credits

**Data:** ULB Machine Learning Group — Andrea Dal Pozzolo, Olivier Caelen, Reid A. Johnson, Gianluca Bontempi.
**Methodology:** CRISP-DM (Chapman et al., 2000) · Lean Thinking (Womack & Jones, 1996)
**Tools:** Python · Scikit-learn · Imbalanced-learn · XGBoost · Joblib · Matplotlib · Seaborn

---

## License

This project is licensed under the [MIT License](LICENSE).
© 2026 Jose Lopez Pino

---

*Framework: CRISP-DM + Lean | Methodology: Project-Based Learning (PBL)*

**Jose Lopez Pino**
Industrial Engineer → Data Scientist | Business Focus

[![GitHub](https://img.shields.io/badge/GitHub-joselopezp-181717?style=flat&logo=github)](https://github.com/joselopezp)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-jose--lopez--pino-0077B5?style=flat&logo=linkedin)](https://www.linkedin.com/in/jose-lopez-pino/)
[![Portfolio](https://img.shields.io/badge/Portfolio-applied--ds--portfolio-4CAF50?style=flat&logo=github)](https://github.com/joselopezp/applied-data-science-portfolio)
