# Cardiovascular Age Estimation from BCG and IPG Signals

Can cardiovascular age be estimated from signals a bathroom scale can capture, as well
as it can from clinical tonometry?

This repository holds the analysis behind that question. Ballistocardiography (BCG) and
impedance plethysmography (IPG) are compared against reference applanation tonometry,
the clinical standard, on the same cohort of subjects.

Research project supervised by **Prof. Ramon Casanella**.

---

## What this studies

Applanation tonometry is the clinical reference for assessing arterial stiffness, but it
needs a trained operator and dedicated equipment. Ballistocardiography and impedance
plethysmography can both be captured by a device a person already stands on.

The question is whether those two signals carry enough cardiovascular information to
estimate age, and how they compare against the clinical reference measured on the same
subjects. Age is used as a proxy outcome: it is known exactly for every subject, which
makes it a clean target for testing whether a signal carries cardiovascular information
at all.

Results, figures and statistical tests are in the notebook.

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
advance. Where another estimator scored marginally lower, the notebook reports
RandomForest anyway and notes the difference is within noise. Selecting the winner after
the fact inflates apparent performance; this avoids that.

**Leave-one-out cross-validation** throughout, with **bootstrap 95% confidence
intervals** on every MAE and **permutation tests** for significance. A nested-CV check
confirmed that hyperparameter tuning gained nothing over the default configuration,
which is reported rather than quietly dropped.

**Corruption detection and recovery.** Each subject CSV interleaves single-row metadata,
100 to 240 per-beat scalars, and waveform blocks. Files with known unrecoverable
corruption are listed explicitly. Partially damaged files are recovered where the
undamaged portion is usable, and excluded where it is not.

**Transparent exclusions.** Every record that is dropped is counted and reported, at
both the beat level and the subject level, and error is broken out per source cohort
rather than pooled into a single headline figure.

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
here.** They consist of per-subject physiological recordings (`s1.csv` through `s87.csv`)
and are the property of **Universitat Politecnica de Catalunya (UPC)**, where the study
was conducted. Enquiries about access should be directed to UPC rather than to this
repository.

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
