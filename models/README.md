# Models Directory

This folder contains saved sklearn/joblib artifacts used for prediction.

## KenPom Artifacts

| File | Description |
| --- | --- |
| `kp_only_gbt_raw.joblib` | Final raw GBT model using only KenPom matchup features. |
| `kp_seed_gbt_raw.joblib` | Final raw GBT model using KenPom matchup features plus seed features. |
| `kp_only_feature_columns.joblib` | Ordered feature list expected by `kp_only_gbt_raw.joblib`. |
| `kp_seed_feature_columns.joblib` | Ordered feature list expected by `kp_seed_gbt_raw.joblib`. |

## Non-KenPom Artifacts

| File | Description |
| --- | --- |
| `gbt_base.joblib` | Final GBT model for the original feature pipeline. |
| `gbt_base_noseed.joblib` | Final GBT model for the original feature pipeline without seed features. |
| `gbt_sigmoid_calibrator.joblib` | Sigmoid calibrator for the original seeded model. |
| `gbt_sigmoid_calibrator_noseed.joblib` | Sigmoid calibrator for the original no-seed model. |
| `feature_columns.joblib` | Feature list for `gbt_base.joblib`. |
| `feature_columns_noseed.joblib` | Feature list for `gbt_base_noseed.joblib`. |

Use `predict_target_season_kenpom.py` with `--model kp_only` or `--model kp_seed` to load the KenPom artifacts.
