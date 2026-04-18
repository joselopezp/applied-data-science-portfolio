# Executive Summary — Fraud Detection System

> **Decision supported:** At what classification threshold should a transaction be flagged
> as fraudulent, given a cost ratio of 20x between missing fraud and blocking a legitimate transaction?

---

## Context

Financial institutions face two simultaneous failure modes in fraud detection: undetected
fraud (direct monetary loss) and legitimate transactions incorrectly blocked (customer
friction and churn risk). Standard ML approaches that optimize for accuracy fail silently —
a model that labels every transaction as legitimate achieves 99.83% accuracy while catching
zero fraud. The stakeholders are the Risk/Fraud team (maximize fraud caught), the Customer
Experience team (minimize false blocks), and the CFO (minimize total expected monetary loss).

---

## Key Findings

| Finding | Evidence | Implication |
|---|---|---|
| Accuracy is the wrong metric | Naive classifier scores 99.83% accuracy catching 0% of fraud | AUC-PR and Expected Loss must replace accuracy as evaluation criteria |
| Default threshold (0.5) is suboptimal | Cost ratio 20x (FN=$300, FP=$15) shifts optimal threshold to t*=0.0414 | Threshold calibration by cost function is mandatory — not optional |
| class_weight beats SMOTE | RF + class_weight wins 9-model leaderboard on AUC-PR (val=0.8584) | Synthetic oversampling (SMOTE) does not improve over native class weighting here |
| Labeled data adds significant value | Isolation Forest (unsupervised) achieves lower AUC-PR than all supervised models | Investment in fraud labeling is justified by measurable performance gain |
| Cost calibration drives business value | Model at t*=0.0414 reduces Expected Loss by 79.7% vs naive baseline | Real cost data (not assumed values) would further improve the threshold |

---

## Results

| | Naive baseline | This system |
|---|---|---|
| Fraud caught | 0 of 71 (0%) | 58 of 71 (81.7%) |
| Legitimate blocked | 0 | 28 |
| Expected Loss | USD 21,300 | USD 4,320 |
| **Savings** | — | **USD 16,980 (79.7%)** |

**Model:** Random Forest + class_weight='balanced' · Threshold: t* = 0.0414
**Test set:** 42,721 transactions · 71 fraud cases (0.17% rate)

### Targets vs results

| Metric | Target | Result | Met? |
|---|---|---|---|
| AUC-PR | ≥ 0.85 | 0.7861 | ❌ |
| Recall (fraud) | ≥ 0.80 | 0.8169 | ✅ |
| Precision (fraud) | ≥ 0.70 | 0.6744 | ❌ |
| Expected Loss vs naive | < baseline | −79.7% | ✅ |

**On unmet targets:** AUC-PR (0.7861 vs 0.85) reflects the difficulty of anonymized PCA
features — addressable with XGBoost tuning. Precision (0.6744 vs 0.70) is the expected
consequence of the 20x cost ratio: the business explicitly prioritized catching fraud over
avoiding false alarms. Both gaps are documented, not hidden.

---

## Recommendation

**Deploy as a first-line fraud filter with human review for all flagged transactions.**
The system catches 81.7% of fraud at 79.7% lower cost than doing nothing, while keeping
false positives low enough for manual review (28 per test window). The cost ratio parameter
(currently assumed at 20x) should be calibrated against real institutional loss data to
maximize threshold accuracy before production deployment.

---

## Limitations & Assumptions

- Dataset covers only 2 days in 2013 (European cardholders) — fraud patterns have evolved
- Features V1–V28 are anonymized PCA components — no domain feature engineering possible
- cost_fn=$300 and cost_fp=$15 are assumed values — not derived from real loss records
- Model not tested on real-time streaming data
- No automated retraining pipeline implemented

---

## Next Steps

| Action | Owner | Timeline |
|---|---|---|
| Calibrate cost_fn and cost_fp with real institutional loss data | Risk / Finance team | Sprint 1 |
| Implement human review queue for flagged transactions | Operations team | Sprint 1 |
| Test XGBoost with hyperparameter tuning (close AUC-PR gap) | ML team | Sprint 2 |
| Retrain model monthly with new transaction data | ML team | Ongoing |
| Wrap model in REST API for real-time scoring | Engineering team | Sprint 3 |

---

*CRISP-DM + Lean · applied-data-science-portfolio · April 2026*
*Jose Lopez Pino — Industrial Engineer → Data Scientist | Business Focus*
