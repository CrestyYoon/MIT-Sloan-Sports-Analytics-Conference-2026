# Methods (EN)

## Research question
Can we **measure in‑game momentum** in soccer as a **latent state** that links *current* events to **near‑future outcomes** (within 5 minutes)? If so, does the **state‑space approach** provide **practical predictive gains** and **interpretable driver coefficients** vs. OLS?

## Data & pipeline
1. **Raw data**: StatsBomb Events & 360 (not redistributed; used to engineer features).
2. **Preprocessing (1–6 notebooks)**: event refinement, sequence IDs, defensive shape features from 360, construction of **attack** and **defense** metrics.
3. **Outcomes**: For each event time, aggregate the next‑5‑minute **xT**, **xG**, and **possession share (“hold”)** for **home/away**.
4. **Wide table**: align features and outcomes per event; export `model_wide_table.csv` for transparency.
5. **Holdout**: Global split via **`match_id % 5`** with **remainder=3** as **test** (Train=237, Test=55).

## Models
### OLS baseline
Event‑level regression of outcome on features (per panel). We report coefficients, t/p, and OOS RMSE/MAE.

### State‑Space Model (SSM)
- **Formulation**: UnobservedComponents with **local level** and **exogenous features** (the metrics).
- **Per‑match fit** (MLE), then **inverse‑variance pooling** of coefficients across matches,
  with **1% winsorization** and **1% variance floor** for stability.
- **A‑1: Possession‑aware observation masking (predict‑only across off‑possession)**  
  When focal team does **not** possess the ball, we **mask observations** so the
  **Kalman filter performs the transition (predict) step only** and **skips the update**.
  This prevents mixing team momentum across possession switches while keeping the
  state continuous in time.
- **Pooled inference**: after per‑match fits, we compute pooled `coef`, `SE`, `z`, `p`, and match counts.
- **Evaluation**: OOS RMSE/MAE computed with **μ‑adjusted predictions** (`ŷ = μ_train + Xβ`),
  plus **SNR = Var(ŷ)/Var(y−ŷ)` for interpretability.

> We emphasize RMSE/MAE for **practical** comparison. **DM tests** are included for completeness but may be extreme at event granularity.

### Code availability & verification scope

We release three feature‑engineering notebooks (`1_Event_Refine.ipynb`, `3_Sequence_Line.ipynb`,
`5_Attack_metric.ipynb`) and **all derived artifacts** required to verify the paper’s numbers.
The full state‑space training/evaluation notebook (`7_Analysis.ipynb`) and defense‑side
engineering templates remain private. This choice preserves deployment IP while keeping
the study **auditable**.

**What reviewers can verify without the private code**
1) **Holdout & counts** — Confirm the split key and sample sizes in the summary files.  
2) **OOS metrics** — Recompute RMSE/MAE per panel from `oos_predictions_residuals.csv`
   and compare to `final_evaluation_summary.csv`.  
3) **Pooled inference** — Recreate coefficient/z/p using `ssm_pooled_all.csv`
   (inverse‑variance pooling; variance floor & winsorization as documented).  
4) **Signal snapshot** — Inspect |z| quantiles and SNR via `ssm_snapshot_flat.csv`.  
5) **Feature names** — Cross‑check the columns used by the models with
   `attack_features.csv` and `defense_features.csv`.

This provides full **result reproducibility** (numbers in the paper) without disclosing
the end‑to‑end training pipeline.


## Reproducibility (without code)
With the included CSVs, referees can:
- **Recreate OOS errors** per panel from `oos_predictions_residuals.csv`.
- **Verify pooled coefficients/inference** with `ssm_pooled_all.csv`.
- **Inspect z‑score distributions & SNR** via `ssm_snapshot_metrics.csv`.
- **Audit the split** with `split_table.csv` and `split_info.json`.

## Known considerations
- **Large z‑scores** arise because of small standard errors given large N; do not over‑interpret individual p‑values.
- **Defense panels** include both `target=self` and `target=opp` by design; combined views are reported.
- **Hold (possession share)** can be noisier; we stabilized through winsorization and variance floors.


## Interpretation & application
- Coefficients indicate **direction and relative importance** of event‑level drivers.
- OOS metrics indicate **practical utility** (e.g., defense‑xT/‑xG showed consistent gains).
- Momentum is **detectable but modest** (~1–5% variance signal), aligning with intuition that
  game flow is multi‑factor and heterogenous across matches.



### LLM Disclosure
This project used a large language model (LLM) to help with drafting documentation, code refactoring, and checklist generation. All modeling decisions, data curation, and validation were made by the author. The LLM did not have access to private data and did not run analyses; every script and output was executed locally and verified by the author.
