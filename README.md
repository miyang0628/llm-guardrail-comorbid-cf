# LLM-Guided Counterfactual Explanation for Comorbid Chronic Disease Risk
## Age × Sex Stratified Framework for Health Insurance Applications

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Overview

This repository contains the full analysis code for the paper:

> **"LLM-Guided Counterfactual Explanation for Comorbid Chronic Disease Risk in Health Insurance: An Age × Sex Stratified Framework"**
> *Submitted to Communications for Statistical Applications and Methods (CSAM)*

The paper proposes a three-stage hybrid framework that generates **guideline-constrained, personalized health intervention pathways** for policyholders with comorbid hypertension and diabetes. The methodological contribution lies in the integration of:

1. **Age × sex cross-stratified XGBoost** risk classification (4 independent models)
2. **GPT-4o-mini clinical guardrail agent** — dynamically generates patient-specific permitted ranges to block physiologically impossible outputs and comorbidity-specific intervention conflicts
3. **DiCE counterfactual explanation** — produces structurally diverse, guideline-consistent recourse candidates within the guardrail-constrained search space

> **Important note:** The XGBoost models serve as the statistical foundation for counterfactual search, not as the primary contribution. The framework is evaluated on cross-sectional survey data (KNHANES); generated pathways represent model-predicted risk-state transitions, not causally validated treatment prescriptions.

---

## Data

**Korea National Health and Nutrition Examination Survey (KNHANES) 2020–2024**
- Administered by the Korea Disease Control and Prevention Agency (KDCA)
- Raw data available at: [https://knhanes.kdca.go.kr](https://knhanes.kdca.go.kr)
- Due to data use agreement, raw `.sas7bdat` files are **not included** in this repository
- Place the five annual files (`hn20_all.sas7bdat` – `hn24_all.sas7bdat`) in the `data/` directory before running the notebooks

**Analytic sample:** 6,339 adults aged 40+ after exclusion of records with missing values on modeling variables

---

## Repository Structure

```
llm-cf-comorbid-age-stratified/
│
├── data/                          # Raw KNHANES files (not included)
├── notebooks/
│   ├── 01_preprocessing.ipynb         # Data merging & feature engineering
│   ├── 02_risk_classification.ipynb   # XGBoost 4-model training & evaluation
│   ├── 03_guardrail_agent.ipynb       # GPT-4o-mini guardrail generation (12 cases)
│   ├── 04_counterfactual.ipynb        # DiCE CF generation + stepwise fallback
│   ├── 05_descriptive_analysis.ipynb  # Stratification effect & CF pathway analysis
│   ├── 06_statistical_tests.ipynb     # Hypothesis tests & hallucination comparison
│   ├── 07_governance_check.ipynb      # G1–G4 guardrail governance validation
│   └── 08_model_validation.ipynb      # Calibration, SHAP, stratification test
├── results/
│   ├── tables/                        # CSV, PKL, JSON outputs
│   └── figures/                       # PNG, PDF, TIFF (600 dpi)
├── .env                               # API keys (not committed)
├── requirements.txt
└── README.md
```

---

## Notebook Execution Order

Run notebooks sequentially (01 → 08). Each notebook loads outputs from the previous step.

| Notebook | Key Output |
|---|---|
| 01_preprocessing | `knhanes_comorbid_2020_2024.csv` |
| 02_risk_classification | `model_{group}.pkl`, `model_performance.csv` |
| 03_guardrail_agent | `guardrail_ranges.json` |
| 04_counterfactual | `counterfactuals.json`, `cf_summary.csv` |
| 05_descriptive_analysis | `stratification_effect.csv`, `cf_pathway_analysis.csv`, `cf_diversity.csv` |
| 06_statistical_tests | `kruskal_wallis_results.csv`, `hallucination_comparison.csv` |
| 07_governance_check | `governance_results.csv`, `governance_stats.csv` |
| 08_model_validation | `calibration_results.csv`, `shap_importance.csv`, `stratification_test.csv` |

---

## Environment Setup

```bash
# Create conda environment
conda create -n diceml python=3.10
conda activate diceml

# Install dependencies
pip install -r requirements.txt
```

**API key setup** — create a `.env` file in the project root:

```
OPENAI_API_KEY=sk-...
LLM_MODEL=gpt-4o-mini
```

---

## Requirements

See `requirements.txt` for the full list. Key packages:

| Package | Version |
|---|---|
| xgboost | 2.0.3 |
| dice-ml | 0.11 |
| openai | 1.0+ |
| shap | 0.44+ |
| optuna | 3.x |
| scikit-learn | 1.x |
| pyreadstat | 1.x |
| python-dotenv | 1.x |
| statsmodels | 0.14+ |
| scipy | 1.11+ |

---

## Key Results

| Metric | Value |
|---|---|
| Post-guardrail G1–G4 compliance (all groups) | **100%** |
| Pre-guardrail any-violation rate | 81.25% |
| G2 Energy floor improvement (McNemar) | Pre 40.4% → Post 100%, p < 0.001 |
| G3 Conflict prevention improvement | Pre 70.2% → Post 100%, p < 0.001 |
| Fallback rate (30-patient cohort per group) | 0–3.3% |
| CF diversity: High / Medium / Low | 4 / 7 / 1 (out of 12 cases) |

---

## Limitations

- KNHANES is a **cross-sectional** survey. Generated recourse candidates represent model-predicted risk-state transitions under guideline-based constraints, not causally validated treatment effects.
- LLM-derived guardrail values have not undergone external clinical validation. Review by endocrinologists and dietitians is required before real-world deployment.
- The framework has not been evaluated for fairness across socioeconomic or ethnic subgroups.

---

## Citation

```bibtex
@article{yang2025llm,
  title   = {LLM-Guided Counterfactual Explanation for Comorbid Chronic Disease Risk in Health Insurance},
  author  = {Yang, Munil and Chun, Heuiju},
  journal = {Communications for Statistical Applications and Methods},
  year    = {2026},
  note    = {Under review}
}
```

---

## License

MIT License. See [LICENSE](LICENSE) for details.
