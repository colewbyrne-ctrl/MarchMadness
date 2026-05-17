# Reports Directory

This folder contains generated model evaluation tables, diagnostics, and prediction outputs.

## Primary Presentation Files

| File | Description |
| --- | --- |
| `baseline_scores_kenpom.csv` | Logistic regression baseline metrics for KenPom features. |
| `gbt_kp_scores.csv` | Rolling-validation GBT model search results. |
| `gbt_kp_best_params.json` | Best GBT hyperparameters selected from rolling validation. |
| `calibration_scores_kenpom.csv` | Raw, sigmoid, and isotonic probability calibration comparison. |
| `permutation_importance_kenpom.csv` | Feature importance measured by logloss degradation after permutation. |
| `feature_ablation_kenpom.csv` | Feature-family removal experiments. |
| `game_by_game_results_kenpom.csv` | Deterministic bracket prediction output from the KenPom model. |

## Data Quality Files

| File | Description |
| --- | --- |
| `kenpom_match_report.csv` | Summary of KenPom-to-Kaggle team matching. |
| `kenpom_missing_matchups.csv` | Matchups missing required KenPom fields. |
| `kenpom_unmatched_teams.csv` | KenPom team names that were not mapped to a Kaggle `TeamID`. |
| `kp_feature_missingness.csv` | Missingness summary for KenPom features. |

## Split Files

| File | Description |
| --- | --- |
| `splits.json` | Time-aware splits for the non-KenPom pipeline. |
| `splits_kenpom.json` | Time-aware splits for the KenPom pipeline. |

For the narrative interpretation of these outputs, see `docs/RESULTS_SUMMARY.md`.
