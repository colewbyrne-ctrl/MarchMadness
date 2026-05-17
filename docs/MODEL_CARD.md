# Model Card

## Model Purpose

This model predicts NCAA men's basketball tournament game outcomes. It is intended for bracket analysis, probability ranking, and exploratory comparison of team-strength signals. It is not designed for betting advice or guaranteed winner selection.

## Model Variants

The KenPom pipeline trains two primary model variants:

| Variant | Description |
| --- | --- |
| `kp_only` | Uses KenPom matchup feature differences and absolute differences. |
| `kp_seed` | Uses the same KenPom features plus tournament seed-number difference features. |

The final saved KenPom artifacts are:

| Artifact | Purpose |
| --- | --- |
| `models/kp_only_gbt_raw.joblib` | Raw gradient-boosted tree model without seed features. |
| `models/kp_seed_gbt_raw.joblib` | Raw gradient-boosted tree model with seed features. |
| `models/kp_only_feature_columns.joblib` | Feature list for the `kp_only` model. |
| `models/kp_seed_feature_columns.joblib` | Feature list for the `kp_seed` model. |

## Training Data

Training rows are historical NCAA tournament games transformed into pairwise matchups:

- `TeamA`, `TeamB`
- target `y`, where `1` means `TeamA` won
- feature differences such as `kp_adj_em_diff`
- absolute feature differences such as `abs_kp_adj_em_diff`
- optional seed features for the `kp_seed` model

The KenPom split file uses seasons 2002 through 2024, excluding 2020 because there was no NCAA tournament.

## Validation Design

The model uses time-aware validation:

- rolling validation seasons: 2008 through 2017
- locked final test seasons: 2018, 2019, 2021, 2022, 2023, 2024
- training seasons for final test: 2002 through 2017

This design keeps later tournaments out of earlier validation folds.

## Model Type

The main model is `sklearn.ensemble.HistGradientBoostingClassifier` with log-loss optimization.

Best KenPom GBT hyperparameters:

```text
learning_rate=0.03
max_iter=300
max_depth=3
min_samples_leaf=50
l2_regularization=1.0
random_state=42
early_stopping=False
loss=log_loss
```

## Evaluation Snapshot

Locked final test set, 401 games:

| Model | Probability Method | Logloss | Accuracy |
| --- | ---: | ---: | ---: |
| `kp_only` GBT | raw | 0.6016 | 0.6933 |
| `kp_only` GBT | sigmoid | 0.6566 | 0.6808 |
| `kp_only` GBT | isotonic | 2.5300 | 0.6883 |
| `kp_seed` GBT | raw | 0.6011 | 0.7007 |
| `kp_seed` GBT | sigmoid | 0.6558 | 0.6808 |
| `kp_seed` GBT | isotonic | 2.0237 | 0.7007 |

The raw `kp_seed` GBT has the best listed GBT accuracy and logloss among the calibrated GBT variants.

## Baseline Context

The logistic regression baselines are strong:

| Feature Set | Model | Logloss | Accuracy |
| --- | --- | ---: | ---: |
| `kp_only` | logistic L1 | 0.5839 | 0.6908 |
| `kp_only` | logistic L2 | 0.5863 | 0.6883 |
| `kp_seed` | logistic L1 | 0.5873 | 0.6933 |
| `kp_seed` | logistic L2 | 0.5903 | 0.7007 |

Because the best logistic baseline has lower test logloss than the GBT, it should remain part of any final model comparison or ensemble discussion.

## Most Important Features

Permutation importance on the KenPom model identifies adjusted efficiency margin as the dominant feature:

| Model | Feature | Mean Logloss Delta |
| --- | --- | ---: |
| `kp_only` | `kp_adj_em_diff` | 0.0752 |
| `kp_only` | `kp_adj_oe_diff` | 0.0236 |
| `kp_only` | `kp_adj_de_diff` | 0.0150 |
| `kp_only` | `abs_kp_adj_em_diff` | 0.0125 |
| `kp_only` | `kp_to_pct_diff` | 0.0108 |

## Limitations

- Historical tournament games are a small sample relative to the randomness of single-elimination basketball.
- KenPom name matching depends on clean source data and manual overrides.
- Injuries, late-season roster changes, travel, matchup-specific tactical details, and market information are not modeled directly.
- The bracket simulator chooses the higher-probability winner at each step, so it produces a deterministic bracket rather than a distribution of possible brackets.
- Strong baselines outperform or match the GBT on some metrics, so model selection should not rely on complexity alone.

## Intended Use

Use this model for:

- bracket probability exploration
- feature importance study
- comparing seeds, KenPom metrics, and regular-season signals
- reproducible NCAA tournament modeling practice

Do not use this model as a sole source for financial decisions.
