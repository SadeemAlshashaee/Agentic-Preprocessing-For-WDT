# Step 23 frozen final-test protocol
Written 2026-11-23T15:01:30 BEFORE attack_4 network data or phy_att_4 data were read.

## Data
- Training: phy_att_1, phy_att_2, phy_att_3 and phy_norm (all development data). Test: phy_att_4, once.
- Evaluation rows: seconds that have network data by exact-second join (the development protocol). 
  LSTM: 10-row sequences whose rows all have network data. The raw physical arm is also scored on all rows.
- The run goes ahead as declared whatever attack_4's coverage turns out to be; the covered row count is reported.

## Arms (every configuration below is reported; none is chosen after seeing attack_4)
| Model | Primary (chosen by the frozen rule on development folds) | Reference arms | Secondary (declared, exploratory) |
|---|---|---|---|
| XGBoost | A3: physical + 14 raw per-second network columns | A1: raw physical (all rows and covered rows); A2: physical + packets/s + mean packet size | agent's full fusion plan (Step 21 plan_all) |
| LSTM | A3: net_scaled_cc (physical + 14 network columns, min-max scaled) | A1: raw physical (unscaled, all rows) and phys_scaled_cc; A2: physical + 2 minimal columns, min-max scaled (LSTM cannot train on unscaled packet counts) | robust-scaled network columns (post-hoc observation, Step 22) |
| Qwen3-32B | Q_raw (no component passed the rule) | none | Q_raw_netmin (raw + 2 network numbers), Q_ref_net (reference summary + network) |
- Models and settings exactly as in development: XGBoost 300 trees depth 5 lr 0.1, balanced weights; LSTM Step 16 (hidden 64, 15 epochs,
  10-row sequences); seeds 0, 1, 2. Qwen: temperature 0, thinking off, 75 attack + 75 normal covered rows, decision cutoff 50, Step 17 prompts.
- No tuning, no threshold calibration, no retraining after the test. Decision rule for the binary call: argmax class is not normal (Qwen: score >= 50).

## Metrics
- All models: ROC-AUC (attack vs normal), recall at 5% false alarms, recall, specificity. XGBoost and LSTM additionally: precision, F1, PR-AUC, balanced accuracy, 3-class macro-F1.
- Qwen's balanced subsample inflates precision, accuracy, F1 and PR-AUC; those are not compared across models.
- Uncertainty: seed mean and SD for XGBoost/LSTM; block-bootstrap 95% interval (60-second blocks, 1,000 resamples) for ROC-AUC and recall at 5% false alarms.
- A single test acquisition cannot support the KEEP/DROP rule, so comparisons between arms are descriptive, with intervals.

## One-time rule
- The final-test cell is run once. All result files are written before anything is printed.
- A rerun is allowed only if the run crashes before any result file is written, and any such rerun is disclosed.
- If it finishes, the numbers stand; a later-found bug is reported as an addendum and the original stays primary.
- A rehearsal switch runs the identical code with a development fold standing in for phy_att_4; it never reads phy_att_4.

## Amendment 1 (written 2026-11-24T14:15:16, before any attack_4 label or feature value was read)
- Step 23a showed attack_4's network capture covers only about 36% of the physical run (first ~10 minutes), as for phy_att_1. The run proceeds as declared on the covered rows.
- Qwen subsample sizes are 'up to' 75 attack and 75 normal covered rows (as in Step 17).
- If the covered rows contain fewer than 20 rows of either class, ROC-AUC and related metrics for the networked arms are reported as undefined; the raw all-rows arm is still reported; nothing else changes.
- Secondary LSTM arm (robust scaling) is scored on its own eligible sequences (ratio and entropy defined), as in Step 22.
- Bootstrap blocks are 60 consecutive rows of the acquisition.
- Stage rule: a stage (XGBoost, LSTM, Qwen) that crashes before saving its own result file may be run again; a finished stage is never run again.
