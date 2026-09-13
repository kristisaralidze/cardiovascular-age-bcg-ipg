# Cardiovascular Age Estimation from BCG and IPG Signals

Can cardiovascular age be estimated from signals a bathroom scale can capture, as well
as it can from clinical tonometry?

This repository holds the analysis behind that question. Ballistocardiography (BCG) and
impedance plethysmography (IPG) are compared against reference applanation tonometry,
the clinical standard, on the same cohort of subjects.

Research project supervised by **Prof. Ramon Casanella**.

---

## Result

**A bathroom scale performed as well as the clinical device.**

On the common cohort (n = 67), scale-derived BCG estimated age to within 9.39 years,
against 8.81 years for reference applanation tonometry, the clinical standard method.
A paired Wilcoxon test cannot separate them (p = 0.089). Both clear the mean-age
baseline of 10.51 years, and both clear a demographics-only floor, so the signal is
cardiovascular rather than body size.

Age estimation from these signals is a hard problem, and the reference column is the
benchmark to read every other row against.

| Model | n | MAE (years) | 95% CI | R² | permutation p |
|---|---|---|---|---|---|
| Mean-age baseline | 67 | 10.51 | | 0.000 | |
| Demographics only | 67 | 10.78 | 9.21 to 12.49 | -0.033 | |
| **Reference tonometry** | 67 | **8.81** | 7.25 to 10.35 | +0.222 | 0.0099 |
| **Scale-BCG** | 67 | **9.39** | 7.81 to 11.16 | +0.131 | 0.0099 |
| Scale-BCG, full valid cohort | 71 | 9.34 | 7.85 to 10.91 | +0.172 | |
| IPG | 67 | 9.81 | 7.65 to 12.10 | +0.049 | 0.0396 |
| BCG and IPG combined | 67 | 10.09 | 7.97 to 12.20 | +0.040 | 0.069 |

Paired Wilcoxon signed-rank, Bonferroni-corrected at 0.05/3 = 0.0167:

- Reference vs BCG: p = 0.089, indistinguishable
- Reference vs IPG: p = 0.795, indistinguishable
- BCG vs IPG: p = 0.041, indistinguishable at the corrected threshold

The BCG signal is driven by fiducial timings, chiefly `bcgw_IJ_ms`, the I-to-J interval.

Combining BCG and IPG did not improve on either signal alone, so the two are reported
separately.

---

## How the analysis is structured

**A staged ladder, not a single model.** Stage 0 establishes a demographics-only floor
(BMI, gender, height, weight). Every later stage is reported as *incremental value over
that floor*, so an apparent signal that is really just body size cannot be mistaken for
a cardiovascular one. Demographics alone barely beat the mean-age baseline, which is
what makes the later gains interpretable.

**Quality thresholds derived from the data, not chosen by hand.** Heart-rate and
pulse-transit-time limits come from IQR and MAD fences over the pooled beat
distribution, intersected with physiological guardrails. The notebook states the reason
plainly: hand-picking limits "would make the analyst a hidden source of bias".

**The pre-specified model is reported even when it loses.** RandomForest was fixed in
advance. In several stages ExtraTrees scored marginally lower, and the notebook reports
RandomForest anyway, noting the difference is within noise. Selecting the winner after
the fact inflates apparent performance; this avoids that.

**Leave-one-out cross-validation** throughout, with **bootstrap 95% confidence
intervals** on every MAE and **permutation tests** for significance. A nested-CV check
confirmed hyperparameter tuning gained nothing over the default configuration
(8.88 vs 8.81 years), which is reported rather than quietly dropped.

**Corruption detection and recovery.** Each subject CSV interleaves single-row metadata,
100 to 240 per-beat scalars, and waveform blocks. Files with known unrecoverable
corruption are listed explicitly. Partially damaged files are recovered where the
undamaged portion is usable, and excluded where it is not.

**Transparent exclusions.** 82 subject records loaded, 72 usable, 10 rejected, with
per-cohort counts reported (db1 n = 29, db2 n = 42) and per-cohort error broken out
(db1 MAE 7.80, db2 MAE 10.40).

## Features

Each subject reduces to one feature vector from three families: beat-aggregate
statistics, waveform morphology, and BCG fiducial timings (`IJ_med`, `bcgw_IJ_ms`,
`bcgw_J_ms`, `bcgw_IJ_amp`, `AJ_med`, `RJ_med`).

## Reproducibility

A single random seed is fixed at the top of the notebook, and the software environment
is pinned and printed on every run:

```
Python 3.13.12 | numpy 2.4.6 | pandas 3.0.3 | sklearn 1.9.0 | scipy 1.17.1
CatBoost catboost.core | Optuna 4.9.0
```

Outputs are committed, so the analysis and every figure can be read without access to
the underlying recordings.

## Data availability

**The subject recordings are not included in this repository and are not redistributable
here.** They consist of per-subject physiological recordings (`s1.csv` through
`s87.csv`) collected under academic supervision. Enquiries about access should be
directed to the supervising institution rather than to this repository.

To run the notebook, place the per-subject CSV files in a directory and point
`<DATA_DIR>` at it.

## Running it

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter lab notebooks/
```

## Authors

- Kristine Saralidze
- Areeba Ahmad
- Savelii Bogdanov

Supervised by Prof. Ramon Casanella.

## Licence

Copyright (c) 2026 the authors. All rights reserved for the code in this repository.
