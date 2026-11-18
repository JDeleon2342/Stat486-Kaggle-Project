## Checkpoint: Evidence and Interpretation (November 19, 2026)

**Team:** Jose De Leon, Rylan Van Komen, Nick Austin
**Route:** B (Kaggle competition, two pipelines)
**Date:** November 17, 2026

**Code:** [`working_investigation_python.ipynb`](working_investigation_python.ipynb) (Pipeline 2, XGBoost). Pipeline 1 is still [`kaggle_pipeline.ipynb`](kaggle_pipeline.ipynb) with outputs [`validation_metrics.csv`](validation_metrics.csv) and [`holdout_predictions.csv`](holdout_predictions.csv).
All tables and figures below are printed in the executed notebook; section numbers refer to its headings. Setting `SAVE_OUTPUTS = True` in the config cell also writes them to `outputs/` as CSV/PNG.

---

### Route B: comparing the pipelines

**What substantive design change did we make?**

| | Pipeline 1 (`kaggle_pipeline.ipynb`) | Pipeline 2 (`working_investigation_python.ipynb`) |
|---|---|---|
| Model | `HistGradientBoostingRegressor` (200 iter, lr 0.05) | `XGBRegressor` / `XGBClassifier`: 300 trees, lr 0.03, `max_depth=3`, `min_child_weight=5`, `subsample=0.8`, `colsample_bytree=0.8`, `reg_lambda=5` |
| Outcome | raw `career_games_started` | **start rate** = games started / (17 x seasons since draft), clipped to [0, 1]; plus a binary "regular starter" (rate >= 0.5) |
| Missing drills | median imputation | left as `NaN`; XGBoost learns where missing values go |
| Features | 13 traditional combine columns + draft year | traditional columns **+ 19 tracking features** built from `combine_tracking.csv`, plus NGS scores and draft capital as reference sets |
| Validation | one 75/25 holdout; CV without a baseline | 5x5 repeated K-fold with the **same 25 folds for every model and both baselines**; Pipeline 1's exact 75/25 holdout; leave-one-draft-class-out |

The outcome change matters. The career file runs through the 2025 season, so the 2023 class has had up to 51 games and the 2025 class only 17. Mean raw starts fall from 12.5 (2023) to 3.4 (2025), while mean start rate is 0.245, 0.235 and 0.202 (Section 2, Figure 1).

The tracking features (Section 3) are one row per player, taken from his best attempt:
- **40-yard dash (418 players):** tracked 10- and 40-yard times, peak speed, peak acceleration, time to peak, and distance to reach 90% of peak.
- **Short shuttle (170) and three-cone (160):** peak speed, acceleration, deceleration, and time moving.
- **Position skill drills (all 510):** peak speed and acceleration z-scored within each drill and averaged per player, plus top speed and number of drills run.

As a sanity check, the tracked 40 time correlates r = 0.904 with the official 40, and tracked 40 peak speed r = -0.978.

**Did results improve or degrade?** Local validation, start-rate target, 25 shared folds (Section 5, Figure 2):

| Model (feature set) | # features | CV MAE | R² | Beats position mean (folds) | Regular-starter AUC (Section 8) |
|---|---:|---:|---:|---:|---:|
| Baseline: global mean | 0 | 0.2502 | -0.018 | 7/25 | — |
| Baseline: position mean | 1 | 0.2457 | 0.005 | — | 0.592 |
| A. Traditional combine | 17 | 0.2352 | 0.010 | 21/25 | 0.599 |
| B. Tracking only | 24 | 0.2443 | -0.060 | 11/25 | 0.606 |
| **C. Traditional + tracking (Pipeline 2)** | 36 | **0.2338** | 0.011 | 21/25 | 0.612 |
| D. C + NGS scores | 39 | 0.1780 | 0.333 | 25/25 | 0.838 |
| E. D + draft capital | 41 | 0.1471 | 0.509 | 25/25 | 0.903 |

(Feature counts include the five position dummies. MAE is in start-rate units, so 0.01 is one percentage point of possible games started.)

Paired comparisons on the same folds (Section 5):
- **Adding tracking (C vs. A):** MAE -0.0014 (SD 0.0075). C is better in only 14 of 25 folds, so the difference is indistinguishable from zero.
- **NGS scores (D vs. C):** MAE -0.0557, better in 25/25 folds.
- **Draft capital (E vs. D):** MAE -0.0309, better in 25/25 folds.

**Bridge to Pipeline 1:** Pipeline 1's exact 75/25 split (`random_state=307`, the same 128 validation players), scored in games started (Section 6):

| Run | Target | MAE | RMSE | R² |
|---|---|---:|---:|---:|
| Pipeline 1: mean baseline | raw starts | 8.296 | 10.878 | 0.000 |
| Pipeline 1: HistGradientBoosting | raw starts | 8.965 | 11.631 | -0.143 |
| XGBoost, Pipeline 1 features | raw starts | 8.150 | 10.869 | 0.002 |
| XGBoost, Pipeline 1 features + tracking | raw starts | 8.507 | 11.395 | -0.097 |
| XGBoost, set C | start rate x games possible | 7.991 | 10.972 | -0.017 |
| XGBoost, set E (+NGS, draft) | start rate x games possible | 4.658 | 7.419 | 0.535 |

The model change alone (same features, same split) moves Pipeline 1 from worse than the baseline to slightly better on MAE (8.150 vs. 8.296). R² is still about 0, so this is "no longer overfitting," not "found signal." On this single holdout, adding tracking hurt (8.507), but over 25 folds it was a wash. That gap is a reminder that one 128-player split is noisy.

**New draft class, leave-one-class-out (Section 7):**

| Model | 2023 | 2024 | 2025 | Mean |
|---|---:|---:|---:|---:|
| Position mean | 0.2440 | 0.2447 | 0.2450 | 0.2446 |
| A. Traditional | 0.2186 | 0.2320 | 0.2519 | 0.2342 |
| C. Traditional + tracking | 0.2246 | 0.2310 | 0.2286 | 0.2281 |
| E. + NGS + draft | 0.1410 | 0.1483 | 0.1613 | 0.1502 |

Tracking helps when the 2025 class is held out (0.2286 vs. 0.2519) and hurts for 2023, so the direction is inconsistent.

**Bottom line:**
- Pipeline 2 improved on Pipeline 1. XGBoost on a start-rate target beats the baselines where Pipeline 1 did not.
- **The improvement comes from the model and target changes, not from the tracking data.** Combine measurements explain about 1% of the variation in start rate. Tracking features add nothing measurable over the traditional combine numbers.
- Large gains appear only when we add information that scouts already summarize: NGS scores and draft slot.

---

### Interpretation and next step

**One informative limitation: the best "combine" predictors aren't really combine measurements.** In the set D model (no draft capital), the top feature by gain is `ngs_final_score` (11.7% of total gain). Fourth is `ngs_college_production_score`, which is college production, not athletic testing. The top tracking feature, `trkSK_n_drills`, is just a count of position drills the player ran (Section 9, Figure 3). It measures participation, not movement quality. A player who runs more drills may be healthier, a stronger prospect, or just at a different position, and none of that is the sensor signal we set out to test.

This affects the conclusion in two ways:
- The jump from C to D (R² 0.011 to 0.333) should not be read as "combine data predicts starts." It says scouting-style summaries that include college production carry most of the information.
- Even the small amount the tracking set contributes may partly reflect participation rather than speed or acceleration. Our "tracking adds nothing" finding is therefore, if anything, generous to tracking.

**Other limitations we found:**
- **The tracked clock starts late.** It starts at the first tracked frame, after the player is already moving. The tracked 10-yard time correlates only 0.47 with the official split, and for 37 of the 418 dash runners no attempt reaches 40 yards from that first frame, so `trk40_time_40yd` exists for only 381 players.
- **Predictions bunch up in the middle.** Out-of-fold predictions for set C mostly fall between 0 and 0.5 regardless of the true rate (Section 10, Figure 4). Absolute error is larger for players who did start (0.257) than for players with zero starts (0.197), so the model mostly fails to pick out the eventual starters.
- **Offensive linemen are hardest to predict.** OL has the largest error (MAE 0.305, vs. 0.187 for WR).
- **Small sample.** With 510 players and up to 36 features, differences between A, B and C are within one fold-to-fold standard deviation (about 0.018 MAE, about 0.06 AUC).

**What the evidence changed in our thinking:** We expected the sensor data to add predictive value beyond stopwatch results. It did not. The tracked measurements largely repeat what the stopwatch already captures (r = 0.90 to 0.98 for the 40). We also learned that Pipeline 1's "worse than baseline" result was partly self-inflicted: raw career totals across unequal careers, plus a model that overfit. With those fixed, combine measurements still have only weak signal.

**What we will test or revise next:**
1. **Ask the question draft capital can't answer.** Predict the *residual* start rate after draft slot (actual minus what a player's draft position predicts). Then test whether combine or tracking features explain who outperforms or underperforms his draft slot. That is the version of the question where sensor data could matter most.
2. **Model each position group separately**, starting with OL and DB, using a few features chosen for that position instead of all 36.
3. **Fix the tracked start time** by detecting motion onset (first frame above a speed or acceleration threshold) and recompute split-time features.
4. **Separate the NGS scores** (athleticism vs. college production vs. final) so pre-draft *physical* information is not mixed with production.
5. **Tune with nested CV** (depth, regularization), so tuning does not leak into the fold scores we compare.

---

### Working code and evidence

**What part of the investigation runs now?** The whole Pipeline 2 path runs end to end in [`working_investigation_python.ipynb`](working_investigation_python.ipynb): raw CSVs, outcome construction, tracking feature engineering, the five-feature-set comparison, the holdout bridge, leave-one-class-out, classification, feature importance, error diagnostics, and a per-player prediction table. The configuration is the "Config" cell in Section 0 (`RANDOM_STATE=307`, `N_SPLITS=5`, `N_REPEATS=5`, `XGB_PARAMS`, `STARTER_THRESHOLD=0.5`, `SAVE_OUTPUTS`). The full run takes about a minute, of which CV is about 26 s.

**Inputs and steps:**
- **Inputs:** `data/combine_results.csv`, `data/player_career_successes.csv`, `data/players.csv` (draft year, draft pick) and `data/combine_tracking.csv`.
- **Steps:**
  1. Join one row per player on `nfl_id`.
  2. Build the start-rate outcome.
  3. Aggregate the ~431k player tracking frames (6,310 drill attempts) into 19 player-level features.
  4. One-hot encode position and fit XGBoost on the shared folds.
- **Data access and setup:**
  - Data comes from the Kaggle competition download and is not committed (`data/` is in `.gitignore`); put the CSVs in `data/`.
  - XGBoost is not in `pyproject.toml`, so the command below adds it for the run only.
  - On macOS, XGBoost needs OpenMP: `brew install libomp`.

**Command to reproduce** (from the repository root):

```bash
uv run --with xgboost --with nbconvert jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=900 working_investigation_python.ipynb
```

Versions used: pandas 3.1.1, numpy 2.5.4, scikit-learn 1.9.1, xgboost 3.4.1, matplotlib 3.11.2.

**What comparison or diagnostic did we run, using which split and metric?**
- **Main comparison:** 5-fold CV repeated 5 times (25 splits, `RepeatedKFold`, `random_state=307`), with the same splits for all five feature sets and both baselines. We score MAE (primary), RMSE and R² on start rate, and add per-fold paired differences.
- **Secondary:** ROC-AUC for "regular starter" on stratified repeated folds.
- **Diagnostics:**
  - Pipeline 1's 75/25 holdout in games-started units.
  - Leave-one-draft-class-out.
  - Gain importance.
  - Out-of-fold predicted vs. actual plot, with error broken down by position, zero vs. non-zero starts, and drafted vs. undrafted.
- **What it shows so far:** see the Route B section above. Pipeline 2 beats the baselines, and tracking adds about nothing over traditional combine measurements.

**Where is the path from raw data to a submission file?**
`working_investigation_python.ipynb` Sections 1 to 11: load (Section 1), prepare the outcome (Section 2) and features (Sections 3 and 4), validate locally (Sections 5 to 8), train and inspect (Sections 9 and 10), and produce predictions (Section 11). Section 11 creates `oof_predictions_pipeline2.csv` (written to `outputs/` when `SAVE_OUTPUTS = True`). It has one row per player, with the out-of-fold predicted start rate from set C and set E and predicted games started from set C.

**What was the local validation result, and why is that validation appropriate?**
- **Result:** for Pipeline 2 (set C), CV MAE is 0.2338 vs. 0.2457 for the position-mean baseline, with R² of 0.011.
- **Why player-level folds:** each player appears exactly once, so no player is in both training and validation.
- **Why repeated, shared folds:** with only 510 players, one split is noisy (the holdout and CV disagreed about tracking). Sharing the same folds across models lets us compare pipelines fold by fold.
- **Why leave-one-class-out:** a new draft class is the realistic use case, and this matches it.

**First valid Kaggle submission:**
- **None yet.** From the NFL/AWS announcements, the Big Data Bowl 2027 appears to be an analytics competition judged on written submissions, not a leaderboard with a hidden test set. That would explain why we never found a test file or sample submission. We will confirm this on the competition's Overview and Rules pages.
- **What stands in for a submission file:** until we confirm the format, the Section 11 out-of-fold prediction table (Pipeline 2, set C) is our closest equivalent. It was produced by this notebook at the commit that adds it.

**Next for the second pipeline:** the residual-on-draft-slot target and position-specific models (items 1 and 2 above). Both will be compared on the same 25 folds.

Source for the competition format: [AWS for M&E blog on the 2027 Big Data Bowl](https://aws.amazon.com/blogs/media/nfl-and-aws-big-data-bowl-will-bridge-the-gap-from-college-to-the-pros/); [NFL announcement](https://www.nfl.com/news/big-data-bowl-returns-powered-by-aws).
