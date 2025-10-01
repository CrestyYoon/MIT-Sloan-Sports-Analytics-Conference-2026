# Measuring In‑Game Momentum in Soccer with State‑Space Models (SSAC 2026)

**Track:** Soccer  
**Submission:** Abstract + open‑source data package (code optional)  
**Holdout rule:** `match_id % 5 == 3 → test`, else train (Train=237, Test=55)

This repository contains the data package and documentation for a study that estimates
**latent, in‑game “momentum”** from event streams using **State‑Space Models (SSM)**
and compares them to baseline **OLS** regressions. We operationalize momentum as the
time‑varying “level” that links current events to **near‑future outcomes** (within five minutes):
**xT**, **xG**, and **ball possession share (“hold”)**.

> **Code policy.** To keep the focus on methods and reproducibility without publishing all implementation know‑how, we provide full data artifacts (CSV) and step‑by‑step **method reproduction guides**. With these, reviewers can recompute key numbers (RMSE/MAE, pooled coefficients, significance groups, DM tests) and verify conclusions.

## Code availability (subset released)

To balance reproducibility with deployment IP, this repo releases **three feature‑engineering notebooks** and all **derived data artifacts**, while keeping the full training/evaluation pipeline private.

**Released notebooks**
- `1_Event_Refine.ipynb` — Clean & normalize events; tag possessions/sequences; write per‑match frames under `data/analysis_final/<match_id>.pkl`.
- `3_Sequence_Line.ipynb` — Derive line‑breaks, Zone‑14/box entries, and sequence geometry; export feature tables used downstream.
- `5_Attack_metric.ipynb` — Build attack‑side features (rates and counts) used by the models.

**Withheld notebooks (IP‑protected)**
- `7_Analysis.ipynb` — State‑space training & evaluation (team‑separated duel‑stream update), OOS scoring, and pooling.
- Defense‑side engineering templates.

**Reproducibility without private code**
All numeric results in the paper can be verified using the released CSVs:
`final_evaluation_summary.csv`, `oos_predictions_residuals.csv`,
`ols_screening_all.csv`, `ssm_pooled_all.csv`, and `ssm_snapshot_flat.csv`.
See *METHODS_EN.md → Reproducibility guide* for step‑by‑step checks.



## Repository Layout (data package)

```
/docs
  ├─ README_EN.md                ← this file
  ├─ DATA_DICTIONARY_EN.md       ← metric & outcome definitions
  └─ METHODS_EN.md               ← methods (pipeline, SSM, A‑1 handling, evaluation)

/outputs
  ├─ final_evaluation_summary.csv  ← OLS vs SSM topline (per perspective/outcome)
  ├─ ols_screening_all.csv         ← OLS screening results
  ├─ ssm_pooled_all.csv            ← SSM pooled screening results (A‑1 applied)
  ├─ oos_predictions_residuals.csv ← Out‑of‑sample predictions & residuals (per panel)
  ├─ ssm_snapshot_metrics.csv      ← Compact per‑panel snapshot (N, z‑summary, SNR)
  ├─ model_wide_table.csv          ← Per‑match, per‑event wide table (features + outcomes)
  ├─ split_info.json               ← holdout key & counts (key=3 → remainder=3 is test)
  └─ split_table.csv               ← match/ split assignment (train/test)

/LICENSE (and LICENSE.txt)       ← data & docs license (CC BY‑NC 4.0 suggested)
```

> **Raw StatsBomb Open Data** are referenced but **not redistributed** here to honor the original license. We include **derived tables** needed to replicate metrics, outcomes, and evaluation.


## What’s inside the outputs

- **final_evaluation_summary.csv** – High‑level comparison (OLS vs SSM) across
  `perspective ∈ {attack, defense}`, `outcome ∈ {xT, xG, hold}`, and
  `target ∈ {self, opp}` (defense panels combine both targets). Includes OOS RMSE/MAE,
  Diebold‑Mariano tests (for reference), and SNR.
- **ols_screening_all.csv** – OLS coefficients, t/p‑values, out‑of‑sample errors.
- **ssm_pooled_all.csv** – SSM pooled coefficients and inference
  (inverse‑variance weighting with 1% winsorization and 1% variance floor).
- **oos_predictions_residuals.csv** – For each panel: event‑level y, y‑hat, residuals,
  and membership (train/test). Use this to recompute RMSE/MAE/DM as desired.
- **ssm_snapshot_metrics.csv** – Panel‑level summary: N matches, |z| quantiles (p50/p90/p99/max),
  and SNR.
- **model_wide_table.csv** – Per‑event “wide” table with the **features actually used** for
  modeling and the **aligned outcomes** (xT/xG/hold over a 5‑minute horizon).
- **split_info.json / split_table.csv** – The exact holdout rule/key and match assignments.


## Reproducing the numbers (without code)

Using the provided CSVs you can:
1. **Verify the split** (`split_table.csv`, `split_info.json`).
2. **Recompute OOS metrics** (RMSE/MAE) per panel from `oos_predictions_residuals.csv`.
3. **Check pooled coefficients and significance** using `ssm_pooled_all.csv`
   (the file includes coefficient, SE, z, p, and match count after pooling).
4. **Recreate snapshot stats** (|z| quantiles, SNR) via `ssm_snapshot_metrics.csv`.
5. **Read the exact metric/outcome definitions** in `DATA_DICTIONARY_EN.md` and the methodological
   details (including A‑1 possession handling and DM interpretation) in `METHODS_EN.md`.


## Limitations & interpretation notes

- **A‑1 possession handling (predict‑only across off‑possession intervals):**
  we block observation updates when the focal team is *not* in possession to avoid mixing team states
  across possession switches; the latent state carries forward via the transition step only.
- **Large z‑scores** can occur because of high event counts and small SEs; we therefore emphasize
  **OOS RMSE/MAE** for practical model comparison.
- **DM tests** at the event granularity can yield extreme statistics; we report them for completeness but
  base practical conclusions on RMSE/MAE.

## Attributions

- **Events & public data:** StatsBomb Open Data (attribution per their policy).  
- **Derived artifacts:** © Authors, shared under **CC BY‑NC 4.0** (see LICENSE).



### LLM Disclosure
This project used a large language model (LLM) to help with drafting documentation, code refactoring, and checklist generation. All modeling decisions, data curation, and validation were made by the author. The LLM did not have access to private data and did not run analyses; every script and output was executed locally and verified by the author.


### Figures & Tables (for abstract)
- `summary_oos_table.png` — one-page key numbers table.
- `oos_rmse_comparison.png` — OOS RMSE comparison (broken y-axis).


### Package Contents (key files)
- `final_evaluation_summary.csv`, `oos_predictions_residuals.csv`
- `ols_screening_all.csv`, `ssm_pooled_all.csv`, `ssm_snapshot_flat.csv`
- `attack_features.csv`, `defense_features.csv`
- `scale_summary.csv`, `zscore_summary.csv`, `split_info.*`
- `METHODS_EN.md`, `README_EN.md` (+ Korean counterparts)
- `summary_oos_table.png`, `oos_rmse_comparison.png`


### Reproduce the Results
1) Create a Python environment with `pandas`, `numpy`, `matplotlib`, and `statsmodels` (versions as in `METHODS_EN.md`).  
2) Place all files under `data/analysis_final/`.  
3) Run the provided scripts to regenerate SSM fits and OLS baselines or use the supplied CSVs for replication of summary numbers.  
4) Use `oos_predictions_residuals.csv` to recompute OOS RMSE and Diebold–Mariano statistics.
