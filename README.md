

---

# **README**

## **Overview**

This repository contains the full experimental framework used to evaluate **MCIR** (Mutual Correlation Impact Ratio) and related dependence-aware attribution baselines.
All code is fully anonymous and self-contained, designed for reproducibility under double-blind peer review.

The repository includes:

* **`mcir_exp.py`** — *Main experiment script.*
  Implements all core experiments reported in the submission, including HouseEnergy-Sim regression, HAR classification (optional), PCIR/MCIR scoring, lightweight fidelity analysis, bootstrap uncertainty, and rank-consistency evaluation.

* **`mcir_new.py`** — *Additional experimental trials.*
  Provides extended analyses, estimator-sensitivity tests, updated block-construction logic, alternate Φ-selection mechanisms, and trials used for ablation and supplementary experiments.

Both scripts use the same underlying MCIR and PCIR formulations and share helper utilities for dependence-aware block construction, conditional MI estimation, bootstrap aggregation, and result-saving.

---

## **Repository Structure**

```
.
│
├── mcir_exp.py        # Main experiment pipeline (core results)
├── mcir_new.py        # Additional trials, ablations, alternative estimators
├── MCIR_for_TMLR_Journal_Submissions.pdf  # Paper draft (ignored in blind review)
│
└── excir_mcir_outputs/   # Auto-generated output directory
       ├── scores/
       ├── bands/
       ├── plots/
       └── logs/
```

All output directories are created automatically when running the scripts.

---

## **1. Main Experiment Script — `mcir_exp.py`**

This file implements the primary experiments accompanying the submitted manuscript:

### **Included Features**

* **HouseEnergy-Sim** regression benchmark (synthetic but structured, with correlated features).
* Optional **UCI HAR** classification benchmark.
* **PCIR** baseline (independent/weak-dependence attribution).
* **MCIR** (mutual-dependence attribution using Gaussian-copula CMI).
* **Lightweight environment construction** for efficiency analysis.
* **Bootstrap uncertainty** (e.g., 5–95% bands).
* **Rank overlays** comparing PCIR vs. MCIR.
* Jaccard@K, Kendall’s τ, Spearman’s ρ metrics (where applicable).
* Automatic saving of:

  * Ranked score CSVs
  * Bootstrap bands
  * Rank overlay figures
  * Lightweight/full-model comparison metrics

### **Usage**

```bash
python mcir_exp.py
```

All output files will appear under `./excir_mcir_outputs/`.

---

## **2. Additional Trials — `mcir_new.py`**

This file extends the main pipeline by including:

### **Additional Contributions**

* Updated **block construction** (more robust cluster merging under correlation variance).
* Patched and sanitized **Φ-selection logic** to guarantee safe indexing and stable conditioning set sizes.
* **Alternative MI estimators**:

  * Gaussian-copula CMI (default)
  * Hybrid discretization-based estimator (ablation)
* **More detailed debugging utilities** for:

  * Redundancy collapse behaviour
  * Estimator stability
  * Lightweight–full agreement
  * Conditioning-set sensitivity
* Extended **HouseEnergy-Sim** trial configurations (varying sample size, dependence strength, estimator type, block size, neighbourhood size).

### **Usage**

```bash
python mcir_new.py
```

Outputs follow the same directory structure but are placed under subfolders indicating the trial type.

---

## **3. Dependencies**

Both scripts rely on standard Python libraries:

* `numpy`, `pandas`
* `scikit-learn`
* `scipy`
* `matplotlib`
* `json`, `os`, `warnings`

Install requirements:

```bash
pip install numpy pandas scikit-learn scipy matplotlib
```

---

## **4. Reproducibility**

Every experiment uses:

* Deterministic random seeds (`seed=` arguments)
* Explicit block construction from empirical correlation structure
* Fixed lightweight fractions (default: 25%)
* Identical model architectures between full and lightweight settings
* Stable copula-Gaussian estimators

All random processes (bootstrap sampling, subsampling, clustering) use seeded RNG.

---

## **5. Extending the Experiments**

To add new datasets:

1. Load or simulate `(X, y)`.
2. Provide a list of feature names.
3. Call:

```python
scores = compute_mcir_all(X, y, feature_names)
```

To add new estimator variants, modify:

* `conditional_mi_gaussian_copula(...)`
* `conditional_mi_hist(...)`
* `mcir_score(...)`

To modify correlated block behavior, edit:

* `build_blocks_from_corr(...)`
* `choose_phi_for_feature(...)`

---

## **6. Citation**

Since this repository is anonymous for review, no author names or affiliations are included.
After acceptance, a full citation entry can be added here.

---

## **7. Contact**

For double-blind peer review, all identifying contact information has been removed.





---

# **8. Dataset Details & Sources**

This repository uses two types of datasets:
(1) **A synthetic but structured benchmark** designed to test dependence-aware attribution, and
(2) **A public real-world dataset** commonly used for feature-attribution benchmarking.

No proprietary or institution-specific data are used.

---

## **8.1 HouseEnergy-Sim (Synthetic Benchmark)**

**Type:** Regression
**Source:** *Fully generated within the scripts (`simulate_houseenergy(...)`) — no external download required.*

### **Purpose**

This dataset simulates realistic **household energy consumption** with strong feature correlations, appliance scheduling patterns, nonlinear HVAC behavior, and weather-driven effects. It is specifically designed to test:

* Dependence-aware attribution under multicollinearity
* Redundancy collapse
* Lightweight vs. full-environment fidelity
* Stability under subsampling and estimator variation

### **Structure**

* ~20 features including:

  * Calendar features (Hour, Weekday)
  * Weather (Outdoor temperature, Solar irradiance)
  * Appliances (TV, Computer, Game console, Lighting, Dishwasher, Dryer, Washing machine, Water heater)
  * HVAC-related variables (Window_open, HVAC_load, Space_heater)
  * Time encodings (Hour_sin, Hour_cos)
* Target: Nonlinear synthetic energy-consumption surrogate.

All correlations arise from shared driving latent variables such as time-of-day and outdoor temperature, making it ideal for evaluating MCIR’s conditional-dependence behavior.

### **How It Is Used**

* Main MCIR results (global attribution)
* Lightweight fidelity evaluation
* Bootstrap uncertainty (200 resamples)
* Estimator sensitivity tests
* Additional trials (`mcir_new.py`)

---

## **8.2 UCI Human Activity Recognition (HAR) Dataset**

**Type:** Binary or multiclass classification (in MCIR only binary is used: class *1 vs non-1*)
**Source (Public):**
UCI Machine Learning Repository — *Human Activity Recognition Using Smartphones Dataset*

**URL:**
[https://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones](https://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones)

*(The repository does not include the dataset; reviewers may optionally place it in a local folder as described below.)*

### **Purpose**

HAR is widely used to evaluate explainability methods on:

* **High-dimensional correlated sensor signals**
* **Nonlinear models such as Random Forest classifiers**
* **Dependence-aware global attribution** when many accelerometer/gyroscope features share common temporal structure

MCIR experiments in HAR validate that dependence-aware ranking properties hold beyond the synthetic dataset and in settings with:

* Signal redundancy
* Periodic structure
* High dimensionality (561 features)

### **Data Requirements**

Expected HAR directory structure:

```
har_root/
   train/
      X_train.txt
      y_train.txt
   test/
      X_test.txt
      y_test.txt
   features.txt
```

In the main script, HAR experiments are optional and only run if a valid path is supplied.

---

## **8.3 Notes on Anonymity and Reproducibility**

* All datasets used in the experiments are either **public** (HAR) or **synthetic** (HouseEnergy-Sim).
* No institution-specific or personal data are included.
* The synthetic generator ensures that results are **reproducible**, requiring only a random seed.
* The repository does not package external data; downloading HAR is optional.

---

## **8.4 Summary Table**

| Dataset             | Type                       | Source            | Dimensionality | Purpose in MCIR Experiments                                                                  |
| ------------------- | -------------------------- | ----------------- | -------------- | -------------------------------------------------------------------------------------------- |
| **HouseEnergy-Sim** | Synthetic regression       | Generated in-code | ~20 features   | Main global attribution evaluation; MCIR vs. PCIR; redundancy collapse; lightweight fidelity |
| **UCI HAR**         | Real sensor classification | UCI ML Repository | 561 features   | High-dimensional benchmark for dependence-aware attribution and classifier explainability    |

---


