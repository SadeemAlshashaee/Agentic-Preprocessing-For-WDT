# Step 14 frozen test plan
Written 2026-11-21T15:37:52 BEFORE any component result was computed.

## Source
Qwen3-32B proposals: step13_sample1_plan.json / step13_sample2_plan.json (unedited, kept as they are).

## Human adjudication (each change tied to a rule)
| Qwen proposed | Tested instead | Rule |
|---|---|---|
| network_available / network_coverage flag | not used; no flag | evidence showed a coverage shortcut (within-acquisition AUC ~0.50) |
| network features only for phy_att_2/3 | one rule for every acquisition; compared on network-covered rows only | acquisition identity cannot be a rule; missing values would act as an implicit flag |
| 10-second non-overlapping windows | causal sliding window ending at each row (past 10 s only): rolling mean, rolling std, 1-step difference | keeps rows one-to-one with the baseline; no leakage across acquisitions |
| falsify if metric 'drops by x' on one fold | kept only if better than baseline by more than the baseline seed SD in ALL three folds | rule fixed by the researchers |
| cleaning: 14/16 constant + correlated pairs (arithmetic inconsistent) | drop features constant in the training fold; drop features with |r|>0.98 to an earlier kept feature (training fold) | arithmetic could not be reconciled; rule is computed, not copied |
| scaling: min-max (sample 2) / no change (sample 1) | min-max fitted on the training fold, tested alone | samples disagreed; test settles it |
| class_handling, normal_session_handling | no change (class weights fitted in training fold; phy_norm in training) | both samples agree |

## Fixed test design
- Folds: leave-one-acquisition-out over phy_att_1, phy_att_2, phy_att_3; phy_norm always in training; phy_att_4 untouched.
- Model: the same XGBoost as Step 9, seeds 0, 1, 2, class weights fitted inside the training fold.
- Primary metric: ROC-AUC attack vs normal (attack = physical_fault or cyberattack). Secondary: macro-F1 at argmax, PR-AUC.
- Verdict per component: KEEP only if (mean variant - mean baseline) > baseline seed SD and > 0 in all three folds. Otherwise DROP.
- Network components are compared against a baseline trained and tested on exactly the same network-covered rows.
- Network feature vocabulary (top Modbus functions) is chosen from pooled development packet counts, no labels.
