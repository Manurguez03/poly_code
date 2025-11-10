# ATTEND1: Upset Prediction – Class Imbalance Workflow Summary

Date: 2025-11-05
Notebook: `class_imbalance.ipynb`

## Objective

Predict tennis match upsets (`upset_binary`: 1 = upset, 0 = non-upset) under strong class imbalance and time-dependent data, while avoiding leakage and producing calibrated probabilities and decision thresholds suitable for imbalanced classification.

## What we built, step by step

### 1) Setup and imports
- Libraries: pandas, numpy, seaborn, matplotlib, scikit-learn (pipelines, preprocessing, metrics, calibration), and XGBoost.
- Configuration: reproducible seed; data directory set to `c:\poly_code\Cleaned_Data_Betting-odds`.

### 2) Data loading and validation
- Loaded multiple per-year Excel files from `Cleaned_Data_Betting-odds`.
- Filtered out temporary/lock Excel files (prefix `~$`).
- Validated required columns including the target `upset_binary` and core features.
- Parsed dates, enforced dtypes, and added a `year` column from filenames.
- Concatenated all years into a single DataFrame `df_all`.

Notes:
- Years present: 2016–2019 and 2021–2024 (no 2020 data).

### 3) Class imbalance exploration
- Computed overall and per-year class distribution of `upset_binary`.
- Visualized:
  - Class counts (upset vs non-upset).
  - Year-wise upset rate.
- Observation: overall upset rate ≈ 7% (rare-positive problem); relatively stable by year with some drift.

### 4) Time-aware splits (to avoid leakage)
- Chronological splitting by `year`:
  - Train: 2016–2022
  - Validation: 2023
  - Test: 2024
- Verified label distribution per split to ensure imbalance assumptions hold across time.

### 5) Baseline with odds features (sanity/leakage check)
- Logistic Regression including odds-derived features (e.g., `AvgW`, `AvgL`, `prob_winner`, `prob_loser`).
- Result: near-perfect validation metrics (as expected), because odds are closely tied to the upset definition — a tautology/leakage for our target.
- Decision: exclude odds-derived features for a fair, market-agnostic evaluation of our modeling signal.

### 6) Baseline without odds features (market-agnostic)
- Built a preprocessing pipeline:
  - Numeric: imputation + scaling.
  - Categorical: OneHotEncoder (sparse_output=True) + imputation.
- Model: Logistic Regression with `class_weight='balanced'`.
- Evaluation:
  - Predicted probabilities on validation; default threshold 0.5 under-detected upsets:
    - Validation results (no-odds baseline, threshold 0.5)
        PR-AUC (AP): 0.3805
        F1 (upset): 0.3708
        MCC: 0.3827
        Balanced Accuracy: 0.8277
        Confusion matrix:
        [[1934, 501],
        [25, 155]]
  - Performed threshold sweep on validation to maximize F1 for the positive class (upset)
    - Best threshold by F1 (upset): 0.74
  - Applied the best threshold to the test set (2024) and reported metrics
    - Test results (2024, using tuned threshold 0.74):
        PR-AUC (AP): 0.2315
        F1 (upset): 0.3341
        MCC: 0.2928
        Balanced Accuracy: 0.6866
        Confusion matrix:
        [[2266, 191],
        [84, 69]]


### 8) Boosted model without odds (XGBoost)
- Preprocessed training data to sparse OHE design matrix.
- XGBClassifier with class weighting via `scale_pos_weight = neg/pos` (derived from training split).
- Conservative hyperparameters (compatibility-friendly) without early stopping.
- Validation (2023):
  - Average Precision (PR-AUC): ~0.566
  - Best threshold by F1: ~0.86
  - At tuned threshold: F1 ≈ 0.552, Precision ≈ 0.520, Recall ≈ 0.589, Balanced Acc ≈ 0.774, MCC ≈ 0.518
  - Confusion (val): [[2337, 98], [74, 106]]
- Test (2024) at the tuned threshold:
  - Average Precision (PR-AUC): ~0.545
  - F1 ≈ 0.504, MCC ≈ 0.473, Balanced Acc ≈ 0.760
  - Confusion (test): [[2355, 102], [67, 86]]
- Note: These improved over the logistic no-odds baseline in our runs.

## Key design choices
- Time-aware splits to prevent leakage across years.
- Excluded odds-derived features for primary performance claims (market-agnostic evaluation).
- Addressed imbalance via class weighting (`class_weight` for LR, `scale_pos_weight` for XGB) and threshold tuning on validation.


## Limitations and drawbacks

Data and features
- Odds exclusion trade-off: Excluding odds avoids tautology but also removes a strong signal. If deployment scenario assumes access to odds, ignoring them may understate achievable performance; if not, odds inclusion is invalid for market-agnostic modeling.
- Feature coverage: Limited domain features (e.g., surface-specific Elo/Glicko, fatigue/rest days, travel, head-to-head context, late-line movement) may cap performance.
- Missing 2020 and potential sampling biases across years/tours could cause temporal drift.

Evaluation and validation
- Single split (train: 2016–22, val: 2023, test: 2024) provides a clear story but fewer samples for threshold selection; variance risk remains. Rolling or blocked time-series CV can give more robust estimates.
- Threshold tuning on a single year (2023) may overfit to that year’s prevalence and context; different years may require different operating points.
- Metrics focus on PR-AUC, F1, MCC, balanced accuracy; we did not optimize for expected value/ROI, which is critical for betting use-cases.

Modeling and training
- Class weighting and threshold tuning are effective but can be brittle under prior shift (changing upset rate year to year).
- XGBoost training lacked early stopping and more extensive hyperparameter search; potential performance left on the table.
- No group-aware CV by tournament/player, which can matter if leakage occurs via repeated entities (same players/tournaments across time windows).


## Recommended alternative approach

A pragmatic path that balances realism (odds availability), statistical rigor, and business utility:

1) Residualize against market odds (if odds will be available)
- Treat bookmaker implied probabilities as a strong prior: `p_market` from odds.
- Train a model to predict the residual uplift: `logit(p_true) − logit(p_market)` using only non-odds features (player form, surface-specific Elo, H2H, fatigue, travel, etc.).
- Final prediction: `p_hat = sigmoid(logit(p_market) + uplift_model(features))`.
- Benefits: leverages market efficiency while allowing the model to correct systematic biases where domain features add signal; avoids tautology during learning by never directly predicting the label from raw odds features.


2) Decision policy tuned on business objective
- Tune thresholds on validation to optimize expected value (EV)/ROI, not just F1/MCC.
- Include bet sizing and selection filters (e.g., avoid low edge or high-juice lines).
- Report both statistical metrics (PR-AUC, Brier, MCC) and business metrics (EV, hit rate, drawdown).

3) Lightweight but targeted feature expansion
- Player strength: surface-adjusted Elo/Glicko, recent-form streaks, age/experience.
- Context: rest days, travel distance/time zones, tournament level/round, surface and weather.
- Match-up: head-to-head adjusted for recency and surface; handedness and style.
- Encoding: target or leave-one-out encodings for high-cardinality categorical features with leakage-safe protocols.


