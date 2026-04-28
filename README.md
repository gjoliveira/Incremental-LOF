# Empirical Evaluation of Incremental LOF for Anomaly Detection in Data Streams

*Data Stream Mining – M.IA022, FEUP/FCUP*

---

## Overview

This project empirically evaluates **Incremental Local Outlier Factor (LOF)** as an anomaly detector for data streams, comparing it against **Half-Space Trees (HST)** as a stream-native baseline.

The central question is: *under which conditions is LOF an effective anomaly detector in data streams, and where does it fail?*

All experiments follow a **prequential (test-then-train) protocol**, are repeated over **10 random seeds**, and report **mean ± std** to ensure results are not artefacts of a single run.

---

## Repository Structure

```
.
├── lof.ipynb                  # Main notebook — all experiments
├── datasets/
│   └── creditcard.csv         # Credit card fraud dataset (not included — see below)
└── README.md
```

---

## Requirements

### Python version

Python 3.9+

### Dependencies

Install all dependencies with:

```bash
pip install river scikit-learn numpy pandas matplotlib joblib
```

| Package | Version tested | Purpose |
|---|---|---|
| `river` | ≥ 0.21 | Incremental LOF and Half-Space Trees |
| `scikit-learn` | ≥ 1.3 | ROC AUC, Average Precision, PCA |
| `numpy` | ≥ 1.24 | Numerical operations |
| `pandas` | ≥ 2.0 | Data loading and summary tables |
| `matplotlib` | ≥ 3.7 | All plots |
| `joblib` | ≥ 1.3 | Parallel multi-seed execution |

---

## Dataset

### Synthetic streams

All four synthetic streams (A, B, C, D) are **generated programmatically** inside the notebook. No external files are needed.

### Credit Card Fraud dataset

The real-world experiment (Exp 5) requires the **Credit Card Fraud Detection** dataset from Kaggle:

1. Download from: https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud
2. Place the file at `datasets/creditcard.csv` (relative to the notebook)

The dataset contains 284,807 transactions with 0.172% fraud rate. Features V1–V28 are PCA-transformed by the dataset authors; `Amount` is the only raw feature used (`Time` is dropped).

> **Note:** The notebook caps the stream at 50,000 instances for HST and 3,000 instances for LOF. The full dataset is not needed to reproduce the results.

---

## How to Run

Open and run `lof.ipynb` sequentially from top to bottom. All cells must be run in order — later experiments depend on functions and cached streams defined in earlier cells.

```bash
jupyter notebook lof.ipynb
# or
jupyter lab lof.ipynb
```

### Expected runtimes

| Experiment | What runs | Approx. time |
|---|---|---|
| Exp 0 – HST sweep | 4 configs × 10 seeds on Stream A | ~2 min |
| Exp 1 – Baseline comparison | LOF + HST × 10 seeds + windowed analysis | ~3 min |
| Exp 2A – k under drift | 4 k values × 10 seeds on Stream A | ~4 min |
| Exp 2B – k in stationary stream | 6 k values × 10 seeds on Stream C | ~5 min |
| Exp 3 – Local anomalies | 3 k values + HST × 10 seeds on Stream D | ~4 min |
| Exp 4 – Density drift | LOF k=40 + HST × 10 seeds on Stream B | ~3 min |
| Exp 5 – Credit card (LOF) | Single run, 3,000 instances | ~2 min |
| Exp 5 – Credit card (HST) | Single run, 50,000 instances | <1 min |
| Exp 6 – Runtime profiling | HST only (LOF values hardcoded from prior run) | ~1 min |

> **Warning:** Do not increase LOF stream sizes in Exp 5 or Exp 6 beyond the values in the notebook. Running LOF on >3,000 instances of the credit card dataset takes hours (observed: >600 minutes at 10,000 instances). This is a documented structural limitation of `river`'s `LocalOutlierFactor` implementation, not a bug in the notebook.

---

## Experiments Summary

| # | Name | Dataset | Hypotheses | Key finding |
|---|---|---|---|---|
| 0 | HST Hyperparameter Calibration | Stream A | — | `window_size=100` selected for all HST runs |
| 1 | Baseline: LOF vs HST under drift | Stream A | H2, H3 | HST dominates; LOF never recovers after drift |
| 2A | k sensitivity under drift | Stream A | H4 | All k values fail equally under drift |
| 2B | k sensitivity in stationary stream | Stream C | H4 | Large k strongly better in stable heterogeneous streams |
| 3 | LOF in its ideal scenario | Stream D | H1 | LOF achieves higher Average Precision on local anomalies |
| 4 | Density drift | Stream B | H2 | LOF degrades irreversibly even when distribution restores |
| 5 | Real-world validation | Credit card | H1–H3 | LOF fails due to dimensionality; HST holds at ~0.65 AUC |
| 6 | Runtime & scalability | Stream A + credit card | — | LOF is O(n²); HST is O(1) per instance |

---

## Hypotheses

| ID | Statement | Outcome |
|---|---|---|
| **H1** | LOF outperforms HST on *local* anomalies | Partially supported — higher AP, not higher AUC |
| **H2** | LOF degrades under concept drift | Confirmed |
| **H3** | HST is more stable and recovers faster from drift | Confirmed |
| **H4** | LOF is sensitive to the choice of k | Conditionally confirmed — matters in stable streams, irrelevant under drift |

---

## Known Limitations

- `river`'s `LocalOutlierFactor` has no bounded-buffer option. It stores every observed instance, making it impractical beyond a few thousand instances on real data.
- All synthetic streams are 2-dimensional. Behaviour in higher dimensions may differ due to the curse of dimensionality.
- The credit card dataset uses pre-processed PCA features, limiting the geometric interpretability of distance-based results.
- Detection delay assumes performance recovery to 95% of pre-drift level. Under severe drift this target may never be reached within the stream.

---

## Authors

M.IA022 — Data Stream Mining, FEUP/FCUP