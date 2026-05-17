# NCAA March Madness Prediction Model

This project predicts NCAA men's basketball tournament outcomes using historical tournament results, regular-season team features, seed information, Elo-style strength estimates, and KenPom efficiency metrics.

The strongest branch of the project is the KenPom pipeline. It builds team-season KenPom features, converts tournament games into pairwise matchup rows, trains gradient-boosted tree models, evaluates calibration, saves reusable model artifacts, and produces target-season bracket predictions.

## Project Highlights

- Builds NCAA tournament matchup datasets from Kaggle March Machine Learning Mania data.
- Adds KenPom pre-tournament efficiency features such as adjusted efficiency margin, offense, defense, tempo, shooting profile, rebounding, turnovers, experience, bench, and height.
- Compares simple logistic baselines against gradient-boosted tree models.
- Uses rolling season validation to avoid leaking future tournament information.
- Keeps the final six available seasons as a locked test set: 2018, 2019, 2021, 2022, 2023, and 2024.
- Saves final model artifacts in `models/` and bracket predictions in `reports/`.

## Repository Layout

```text
.
|-- build_data/                     # Core Kaggle data preparation scripts
|-- csv_folder/                     # Source CSV files and KenPom input data
|-- data_processed/                 # Generated parquet datasets
|-- docs/                           # Project writeups and interpretation notes
|-- models/                         # Saved sklearn/joblib model artifacts
|-- reports/                        # Evaluation tables, feature studies, predictions
|-- build_kenpom_team_season.py     # Maps KenPom rows to Kaggle TeamID values
|-- build_train_matchups_kenpom.py  # Creates KenPom matchup training data
|-- create_splits_kenpom.py         # Creates rolling validation/test splits
|-- baselines_kenpom.py             # Logistic regression baselines
|-- train_gbt_kenpom.py             # Gradient boosted tree model search
|-- calibrate_gpt_kenpom.py         # Raw, sigmoid, isotonic calibration comparison
|-- isolate_raw_gbt_kenpom.py       # Fits and saves final KenPom GBT artifacts
|-- predict_target_season_kenpom.py # Generates bracket predictions for a season
```

## Quick Start

Create and activate a Python environment, then install the required packages:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Run the KenPom pipeline:

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
```

Generate target-season bracket predictions:

```powershell
python predict_target_season_kenpom.py --season 2025 --model kp_seed
```

The default prediction output is:

```text
reports/game_by_game_results_kenpom.csv
```

## Current Results Snapshot

The KenPom gradient-boosted tree search selected the same hyperparameters for both `kp_only` and `kp_seed`:

```text
learning_rate=0.03
max_iter=300
max_depth=3
min_samples_leaf=50
l2_regularization=1.0
```

On the locked six-season test set, the raw KenPom plus seed GBT reached:

```text
logloss: 0.6011
accuracy: 0.7007
games: 401
```

The logistic KenPom baselines are competitive and should be treated as important reference models. The best listed baseline is `kp_only` logistic regression with L1 regularization:

```text
logloss: 0.5839
accuracy: 0.6908
games: 401
```

See [docs/RESULTS_SUMMARY.md](docs/RESULTS_SUMMARY.md) for the fuller interpretation.

## Important Notes

- KenPom data is expected at `csv_folder/KenPom.csv`.
- Team-name matching is handled through exact matches, hard-coded exceptions, and `csv_folder/kenpom_name_overrides.csv`.
- The script name `perutation_importance_kenpom.py` contains a typo in the filename; use that spelling unless the file is renamed.
- `calibrate_gpt_kenpom.py` evaluates calibration methods, but the saved KenPom artifacts currently use raw GBT probabilities because raw probabilities performed best on the locked test set.
- The repository includes generated data, reports, and model artifacts, so a reader can inspect outputs without rerunning the whole pipeline.

## Documentation

- [Model Card](docs/MODEL_CARD.md)
- [Data and Pipeline Guide](docs/DATA_AND_PIPELINE.md)
- [Results Summary](docs/RESULTS_SUMMARY.md)
