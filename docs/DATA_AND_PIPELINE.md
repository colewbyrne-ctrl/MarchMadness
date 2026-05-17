# Data and Pipeline Guide

## Inputs

The project uses two main data sources:

| Path | Description |
| --- | --- |
| `csv_folder/` | Kaggle March Machine Learning Mania CSV files. |
| `csv_folder/KenPom.csv` | KenPom pre-tournament team-season metrics. |
| `csv_folder/kenpom_name_overrides.csv` | Manual mapping overrides from KenPom names to Kaggle `TeamID` values. |

Generated parquet files are stored in `data_processed/`. Generated metrics, diagnostics, and predictions are stored in `reports/`.

## Core Kaggle Pipeline

These scripts build the non-KenPom feature set:

| Step | Script | Output |
| --- | --- | --- |
| Load and validate raw files | `build_data/read.py`, `build_data/validate.py` | base parquet files |
| Regular-season long table | `build_data/build_rs_long.py` | `regular_season_long.parquet` |
| Team base statistics | `build_data/build_team_season_base.py` | `team_season_base.parquet` |
| Strength of schedule | `build_data/build_sos.py` | SOS columns joined to team-season data |
| Elo features | `build_data/build_elo.py` | `team_season_elo.parquet` |
| Tournament seeds | `build_data/build_seeds.py` | `seeds.parquet` |
| Full team-season table | `build_data/build_team_season_full.py` | `team_season_full.parquet` |
| Matchup training rows | `build_data/build_train_matchups.py` | `train_matchups.parquet` |

## KenPom Pipeline

The KenPom branch is the most complete branch for presentation:

| Step | Script | Output |
| --- | --- | --- |
| Map KenPom teams to Kaggle IDs | `build_kenpom_team_season.py` | `data_processed/kenpom_team_season.parquet` |
| Create KenPom matchup rows | `build_train_matchups_kenpom.py` | `data_processed/train_matchups_kenpom.parquet` |
| Create time-aware splits | `create_splits_kenpom.py` | `reports/splits_kenpom.json` |
| Evaluate logistic baselines | `baselines_kenpom.py` | `reports/baseline_scores_kenpom.csv` |
| Tune GBT models | `train_gbt_kenpom.py` | `reports/gbt_kp_scores.csv`, `reports/gbt_kp_best_params.json` |
| Compare calibration methods | `calibrate_gpt_kenpom.py` | `reports/calibration_scores_kenpom.csv` |
| Compute permutation importance | `perutation_importance_kenpom.py` | `reports/permutation_importance_kenpom.csv` |
| Run feature ablations | `feature_ablation_kenpom.py` | `reports/feature_ablation_kenpom.csv` |
| Save final raw models | `isolate_raw_gbt_kenpom.py` | `models/kp_*` artifacts |
| Predict a bracket | `predict_target_season_kenpom.py` | `reports/game_by_game_results_kenpom.csv` |

## Reproducible Command Sequence

```powershell
python build_kenpom_team_season.py
python build_train_matchups_kenpom.py
python create_splits_kenpom.py
python baselines_kenpom.py
python train_gbt_kenpom.py
python calibrate_gpt_kenpom.py
python perutation_importance_kenpom.py
python feature_ablation_kenpom.py
python isolate_raw_gbt_kenpom.py
python predict_target_season_kenpom.py --season 2025 --model kp_seed
```

## Feature Families

The KenPom branch creates matchup-level differences for:

| Family | Example Columns | Interpretation |
| --- | --- | --- |
| Core efficiency | `kp_adj_em_diff`, `kp_adj_oe_diff`, `kp_adj_de_diff`, `kp_adj_tempo_diff` | TeamA value minus TeamB value. |
| Rankings | `kp_adj_em_rank_diff` | Difference in KenPom rank-like fields. |
| Style/profile | `kp_efg_pct_diff`, `kp_to_pct_diff`, `kp_or_pct_diff`, `kp_ft_rate_diff` | Shooting, turnovers, rebounding, free throw rate. |
| Roster profile | `kp_experience_diff`, `kp_bench_diff`, `kp_avg_height_diff` | Team composition and rotation signals. |
| Absolute gaps | `abs_*_diff` | Magnitude of separation regardless of direction. |
| Seeds | `seed_num_diff`, `abs_seed_num_diff` | Included only for `kp_seed`. |

## Target-Season Prediction Flow

`predict_target_season_kenpom.py`:

1. Loads a saved model and feature list from `models/`.
2. Loads KenPom team-season rows for the requested season.
3. Loads tournament seeds and team names.
4. Builds every pairwise tournament-team matchup.
5. Creates model features from TeamA-TeamB differences.
6. Predicts `P(TeamA wins)`.
7. Simulates each bracket round by advancing the team with probability at least 0.5.
8. Writes game-by-game bracket results to `reports/game_by_game_results_kenpom.csv`.

## Data Quality Checks

The KenPom mapping step writes helpful diagnostics:

| Report | Meaning |
| --- | --- |
| `reports/kenpom_unmatched_teams.csv` | KenPom names that could not be mapped to a Kaggle `TeamID`. |
| `reports/kenpom_missing_matchups.csv` | Matchup rows missing KenPom information. |
| `reports/kenpom_match_report.csv` | Summary of KenPom matching coverage. |
| `reports/kp_feature_missingness.csv` | Missingness by KenPom feature. |

Review these files when changing `KenPom.csv` or the override table.
