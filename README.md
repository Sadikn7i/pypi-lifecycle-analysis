# The Graveyard of Open Source — PyPI Package Abandonment Study

![Python](https://img.shields.io/badge/Python-3.10+-blue?style=flat-square&logo=python) ![License](https://img.shields.io/badge/License-MIT-green?style=flat-square) ![PyPI](https://img.shields.io/badge/Data-PyPI%20%7C%2096%2C810%20packages-orange?style=flat-square) ![Status](https://img.shields.io/badge/Status-Research%20Paper-purple?style=flat-square)

> *What kills an open source package — and can we predict it before it dies?*

This project is a large-scale empirical study of package abandonment on PyPI. It scrapes, cleans, and analyses ~100,000 Python packages using machine learning, survival analysis, SHAP explainability, dependency network analysis, and revival detection — combining them into a single unified framework for the first time.

---

## Table of Contents

- [Overview](#overview)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Results](#results)
- [Project Structure](#project-structure)
- [How to Run](#how-to-run)
- [Dependencies](#dependencies)

---

## Overview

Over 57% of PyPI packages haven't been updated in more than 3 years. This project asks: **why do packages get abandoned, which ones come back, and does abandonment spread through dependency networks?**

It covers six analytical waves:

| Wave | Focus |
|------|-------|
| 1 | Data collection — async scraping of 100,000 PyPI packages |
| 2 | Cleaning, feature engineering, descriptive statistics (Tables 1–7, Figures 1–4) |
| 3 | Machine learning classification — 7 models, SHAP explainability (Tables 8–11, Figures 5–10) |
| 4 | Survival analysis — Kaplan-Meier curves, Cox regression, risk tiers (Tables 12–13, Figures 11–13) |
| 5 | Revival analysis — detecting packages that came back from the dead (Table 14, Figures 14–15) |
| 6 | Dependency contagion — does abandonment spread through the network? |
| 7 | Robustness checks, sensitivity analysis, related work comparison (Tables 15–18, Figures 16–18) |

---

## Dataset

- **Source:** PyPI public API (`pypi.org/pypi/{name}/json`)
- **Sample:** 100,000 packages drawn randomly (seed=42) from 757K+ on PyPI
- **After cleaning:** ~96,810 packages
- **Abandonment definition:** no release in ≥ 1,095 days (3 years)
- **Abandonment rate:** ~57% of the cleaned sample

### Features engineered per package

| Feature | Description |
|---------|-------------|
| `days_since_last` | Days since most recent release |
| `num_releases` | Total version count |
| `num_dependencies` | Number of declared dependencies |
| `lifespan_days` | Days between first and last release |
| `release_frequency` | Releases per year over lifespan |
| `metadata_score` | Sum of 6 binary metadata completeness flags |
| `has_documentation` | Has a docs URL |
| `has_license` | Has a declared license |
| `has_requires_python` | Specifies Python version compatibility |
| `has_changelog` | Has a changelog link |
| `classifiers_count` | Number of PyPI classifiers |
| `summary_length` | Character length of package summary |
| `first_year` | Year of first release |
| `yanked` | Whether any version was yanked |

---

## Methodology

### 1. Data Collection (Wave 1)
Async scraping with `aiohttp` at 150 concurrent connections. Fetched metadata for 100K packages in ~60–90 minutes. All raw data saved to `pypi_raw.csv`.

### 2. Machine Learning (Wave 3)
Seven classifiers trained and compared:
- Logistic Regression
- Decision Tree
- Random Forest
- Gradient Boosting
- XGBoost
- LightGBM
- Neural Network (MLP)

Model explainability via **SHAP values** — identifying which features actually drive the abandonment prediction, not just what the model weights say.

### 3. Survival Analysis (Wave 4)
- **Kaplan-Meier curves** by metadata quality tier and release frequency
- **Cox Proportional Hazards regression** with time-varying covariates
- Packages stratified into **low / medium / high risk tiers** based on predicted hazard

### 4. Revival Analysis (Wave 5)
Defined revival as: lifespan ≥ 730 days, recently updated (< 365 days ago), ≥ 3 releases, previously classified as abandoned. Sensitivity tested across 9 threshold combinations.

### 5. Contagion Analysis (Wave 6)
Built a directed dependency graph from 30,000 re-fetched packages. Tested whether having abandoned dependencies increases a package's own abandonment probability via log-rank test. Result: trend observed (p = 0.0723), not statistically significant — likely due to sparse graph coverage (79.8% of edges point outside dataset).

### 6. Robustness Checks (Wave 7)
Full vs reduced feature set (dropping `lifespan_days` and `first_year`) across all 7 models. AUC drops were minimal (< 0.01 for top models), confirming result stability.

---

## Results

| Model | ROC-AUC | F1 |
|-------|---------|-----|
| LightGBM | **0.94** | **0.91** |
| XGBoost | 0.93 | 0.90 |
| Random Forest | 0.92 | 0.89 |
| Gradient Boosting | 0.91 | 0.88 |
| MLP | 0.88 | 0.85 |
| Logistic Regression | 0.81 | 0.78 |
| Decision Tree | 0.79 | 0.76 |

**Top SHAP predictors of abandonment:**
1. `days_since_last` (dominant)
2. `release_frequency`
3. `metadata_score`
4. `num_releases`
5. `lifespan_days`

**Survival:** Packages with high metadata scores had 3-year survival rates ~2.4× higher than those with minimal metadata.

**Revival rate:** ~8–14% of packages classified as abandoned showed signs of revival, depending on threshold combination.

---

## Project Structure
```
python-graveyard/
│
├── Python_Graveyard.ipynb     # Full analysis notebook (all 7 waves)
├── pypi_raw.csv               # Raw scraped data (generated, not included)
├── pypi_dep_graph.json        # Dependency graph cache (generated)
│
├── figures/                   # All 18 output figures
│   ├── Figure 01 - ...
│   └── Figure 18 - ...
│
└── tables/                    # All 18 output tables (CSV)
    ├── Table 01 - ...
    └── Table 18 - ...
```

---

## How to Run
```bash
# 1. Clone the repo
git clone https://github.com/Sadikn7i/python-graveyard
cd python-graveyard

# 2. Install dependencies
pip install -r requirements.txt

# 3. Open the notebook
jupyter notebook Python_Graveyard.ipynb
```

Run the waves in order. Wave 1 will take 60–90 minutes to fetch data. All subsequent waves load from the saved CSV.

> **Note:** Set `SAVE_DIR` in each wave cell to your preferred output folder before running.

---

## Dependencies
```
requests
pandas
numpy
matplotlib
seaborn
aiohttp
nest_asyncio
tqdm
scikit-learn
xgboost
lightgbm
lifelines
shap
networkx
```

---

## Author

**Sadik Aden Dirir**
Nagoya University · University of Wolverhampton
[github.com/Sadikn7i](https://github.com/Sadikn7i)
