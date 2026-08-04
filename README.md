# LLM-Guided Counterfactual Explanation for Comorbid Chronic Disease Risk

## Age × Sex Stratified Framework for Health Insurance Applications

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Overview

This repository contains the full analysis code for the paper:

> **"A Guardrail-Augmented Counterfactual Explanation Framework for Comorbid
> Hypertension and Diabetes Risk: An Age × Sex Stratified Approach"**
> *Submitted to Communications for Statistical Applications and Methods (CSAM)*

The paper proposes a three-stage hybrid framework that generates
**guideline-constrained, personalized recourse pathways** for policyholders with
comorbid hypertension and type 2 diabetes. The framework integrates:

1. **Age × sex cross-stratified XGBoost** risk classification (4 independent models)
2. **A layered clinical guardrail** — a GPT-4o-mini agent supplies a
   patient-specific clinical rationale, and a deterministic rule layer
   (Rules A–E) translates the guardrail principles into the permitted ranges
   that constrain the counterfactual search
3. **DiCE counterfactual explanation** — produces structurally diverse,
   guideline-consistent recourse candidates within the guardrail-constrained
   search space

> **Note on the role of the two guardrail components.** The permitted ranges are
> determined by the deterministic rule layer; the language model contributes the
> accompanying clinical rationale rather than the numeric bounds. The reduction
> in physiologically implausible recourse reported here is driven primarily by
> the deterministic component. See `notebooks/03b`, `05d`, `07c` and
> `EVAL_DESIGN_07c.md` for the evaluation that establishes this.

> **Note on the XGBoost models.** These serve as the statistical foundation for
> counterfactual search, not as the primary contribution. The framework is
> evaluated on cross-sectional survey data (KNHANES); generated pathways
> represent model-predicted risk-state transitions, not causally validated
> treatment prescriptions.

---

## Data

**Korea National Health and Nutrition Examination Survey (KNHANES) 2020–2024**

- Administered by the Korea Disease Control and Prevention Agency (KDCA)
- Raw data available at: <https://knhanes.kdca.go.kr>
- Due to the data use agreement, raw `.sas7bdat` files are **not included**
- Place the five annual files (`hn20_all.sas7bdat` – `hn24_all.sas7bdat`) in the
  `data/` directory before running the notebooks

**Analytic sample:** 6,339 adults aged 40+ after exclusion of records with
missing values on modeling variables.

> **Sampling weights.** The current analyses do not incorporate the KNHANES
> complex-survey sampling weights; estimates are therefore not nationally
> representative. Weighted analysis is identified as future work in the paper.

---

## Repository Structure

```
llm-guardrail-comorbid-cf/
│
├── data/                          # Raw KNHANES files (not included)
├── notebooks/
│   ├── 01_preprocessing.ipynb          # Data merging & feature engineering
│   ├── 02_risk_classification.ipynb    # XGBoost 4-model training & evaluation
│   ├── 03_guardrail_agent.ipynb        # GPT-4o-mini reasoning + Rules A–E ranges
│   ├── 03b_guardrail_agent_llm_raw.ipynb   # Captures LLM raw ranges + hard-rule
│   │                                       #   + combined range sets (for ablation)
│   ├── 04_counterfactual.ipynb         # DiCE CF generation + stepwise fallback
│   ├── 05_descriptive_analysis.ipynb   # Stratification effect & CF pathway analysis
│   ├── 05d_final_eval_sensitivity.ipynb    # Hallucination eval, external scoring,
│   │                                       #   range-widening sensitivity
│   ├── 06_statistical_tests.ipynb      # Hypothesis tests & pathway diversity
│   ├── 07_governance_check.ipynb       # Original G1–G4 governance check
│   ├── 07c_governance_honest_eval.ipynb    # External-threshold governance eval,
│   │                                       #   soft/hard split, violation attribution
│   └── 08_model_validation.ipynb       # Calibration, SHAP, stratification test
├── results/
│   ├── tables/                         # CSV, PKL, JSON outputs
│   └── figures/                        # PNG, PDF, TIFF (600 dpi)
├── EVAL_DESIGN_07c.md                  # Evaluation-design rationale (external
│                                       #   scoring, soft vs hard, attribution)
├── .env                                # API keys (not committed)
├── requirements.txt
└── README.md
```

---

## Notebook Execution Order

Run `01 → 08` sequentially for the main pipeline. The revision notebooks
(`03b`, `05d`, `07c`) implement the hallucination-reduction evaluation and the
LLM-only / hard-rule-only / combined ablation; they read `03b`'s output and can
be run after `03b`.

| Notebook                       | Key Output |
| ------------------------------ | ---------- |
| 01_preprocessing               | `knhanes_comorbid_2020_2024.csv` |
| 02_risk_classification         | `model_{group}.pkl`, `model_performance.csv` |
| 03_guardrail_agent             | `guardrail_ranges.json` (LLM reasoning + hard-rule ranges) |
| 03b_guardrail_agent_llm_raw    | `guardrail_ranges_v2.json` (LLM raw / hard-rule / combined) |
| 04_counterfactual              | `counterfactuals.json`, `cf_summary.csv` |
| 05_descriptive_analysis        | `stratification_effect.csv`, `cf_pathway_analysis.csv`, `cf_diversity.csv` |
| 05d_final_eval_sensitivity     | `final_eval_summary.csv`, `final_eval_percf.csv`, `final_eval_feasibility.csv` |
| 06_statistical_tests           | `kruskal_wallis_results.csv`, `cf_diversity.csv` |
| 07_governance_check            | `governance_results.csv`, `governance_stats.csv` |
| 07c_governance_honest_eval     | `governance_07c_summary.csv`, `governance_07c_mcnemar.csv`, `governance_07c_attribution.csv` |
| 08_model_validation            | `calibration_results.csv`, `shap_importance.csv`, `stratification_test.csv` |

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

> **Reproducibility note.** Outputs at `temperature=0` are not always identical
> across runs, consistent with reported LLM non-determinism. Because the permitted
> ranges are fixed by the deterministic rule layer, the final ranges are stable
> even when the language-model rationale varies; the language-model text itself
> is not guaranteed to be identical across runs.

---

## Requirements

See `requirements.txt` for the full list. Key packages:

| Package       | Version |
| ------------- | ------- |
| xgboost       | 2.0.3   |
| dice-ml       | 0.11    |
| openai        | 1.0+    |
| shap          | 0.44+   |
| optuna        | 3.x     |
| scikit-learn  | 1.x     |
| pyreadstat    | 1.x     |
| python-dotenv | 1.x     |
| statsmodels   | 0.14+   |
| scipy         | 1.11+   |

---

## Key Results

Hallucination (physiologically implausible recourse) rates across counterfactual
candidates, scored against **external clinical thresholds** independent of the
range-generation rules (energy floor at the very-low-calorie-diet boundary of
800 kcal/day; sodium–carbohydrate conflict; BMI–weight inconsistency):

| Violation type                 | Pure DiCE | Guardrail (injected) | Guardrail (enforced) |
| ------------------------------ | --------- | -------------------- | -------------------- |
| Energy floor (<800 kcal/day)   | 4.2%      | 8.5%                 | **0.0%**             |
| Sodium–carbohydrate conflict   | 33.3%     | **0.0%**             | 0.0%                 |
| BMI–weight inconsistency       | 8.3%      | **0.0%**             | 0.0%                 |

Range-widening sensitivity (injected stage; primary policy widens only the lower
bounds and keeps the hard-rule upper bounds):

| Policy                         | Energy | Conflict | BMI–weight | Mean CFs/case (min) |
| ------------------------------ | ------ | -------- | ---------- | ------------------- |
| Primary (lower widened)        | 8.5%   | 0.0%     | 0.0%       | 3.92 (3)            |
| Strict (neither widened)       | 0.0%   | 0.0%     | 0.0%       | 3.17 (0)            |
| Widened (both widened)         | 8.5%   | 46.8%    | 6.4%       | 3.92 (3)            |

Other results: all 12 representative cases reached Class 0 (full risk
normalisation); CF diversity 4 High / 7 Medium / 1 Low (of 12); fallback rate
0–3.3% in the 30-patient cohort. Kruskal–Wallis tests indicated heterogeneity in
guardrail permitted ranges across strata for energy, sodium, and weight
(p < 0.05), interpreted as preliminary given n = 3 cases per stratum.

> Exact values may vary slightly across runs due to LLM non-determinism; the
> figures above are from the primary-policy run recorded in
> `results/tables/final_eval_summary.csv`.

---

## Limitations

- KNHANES is a **cross-sectional** survey. Generated recourse candidates
  represent model-predicted risk-state transitions under guideline-based
  constraints, not causally validated treatment effects.
- The permitted ranges are produced by the deterministic rule layer; the present
  results do not establish an independent quantitative contribution of the
  language model to violation reduction.
- Macronutrient floors (protein, potassium, carbohydrate, fibre) are not fully
  guaranteed when scored against external reference intakes.
- The complex-survey sampling weights are not incorporated; estimates are not
  nationally representative.
- Guardrail values have not undergone external clinical validation. Review by
  endocrinologists and registered dietitians is required before real-world
  deployment.
- The framework has not been evaluated for fairness across socioeconomic or
  regional subgroups.

---

## Citation

```bibtex
@article{yang2026guardrail,
  title   = {A Guardrail-Augmented Counterfactual Explanation Framework for
             Comorbid Hypertension and Diabetes Risk},
  author  = {Yang, Munil and Chun, Heuiju},
  journal = {Communications for Statistical Applications and Methods},
  year    = {2026},
  note    = {Under review}
}
```

---

## License

MIT License. See [LICENSE](LICENSE) for details.
