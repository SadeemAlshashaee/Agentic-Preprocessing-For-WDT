# Step 21 frozen plan
Written 2026-11-23T13:41:58 BEFORE any result.

## Adjudication of Qwen's fusion proposals (human)
- alignment_tolerance 2 s [3 of 3] -> ACCEPT: cheap, shared; expected near-null (coverage gain 0.0008 to 0.0024). NOTE: uses up to 2 future seconds, so it is NOT causal (offline only).
- missing_handling ffill_causal 2 s [3 of 3] -> ACCEPT: causal, low risk; affects only short gaps.
- ratio_to_recent_baseline 60 s [3 of 3] -> MODIFY: columns differed per sample; human took the union (net_pkts, net_size_mean/std/max, net_resp_null_share); ratios replace the raw columns. Risk: a long attack becomes its own baseline.
- function_entropy [3 of 3] -> ACCEPT: one extra column, no leakage risk.
- robust_scale_network [3 of 3] -> ACCEPT for LSTM only: XGBoost trees are scale-invariant; Qwen reads text. Tested in Step 22.
- drop_features sport+dport [1 of 3] -> ACCEPT narrow version: sample 3 version; sample 2's four-column version rejected (IP counts likely carry MITM/scan signal).
- rolling_stat mean 5 s [1 of 3] -> REJECT: two samples rejected rolling statistics; physical window test already failed.

## Qwen errors found by human review
- Sample 2, S1: wrote 78.9% for phy_att_1 within-2s coverage; the evidence says 7.89% (tenfold slip).
- Sample 2, S5: cited packet counts 2268 vs 2276 as 'differing'; the numbers are not in the evidence and barely differ.
- Samples 1-3: 'all' / 'net_fn_*' used as column lists, parameters nested incorrectly; normalized mechanically.
- Falsification criteria in sample 3 read in the opposite direction to samples 1-2; the frozen rule decides, not Qwen's thresholds.

## Test design
- Paired baseline: physical + the 14 raw network columns, exact-second join.
- Evaluation rows for EVERY variant: seconds that have network data within 2 s (superset of exact coverage), so all variants are scored on the same rows; baseline keeps NaN where its exact join finds nothing.
- Training rows: the same row set, other development acquisitions + phy_norm.
- Ratio step: median of the previous 60 seconds (>= 10 observed), denominator + 0.01 x training q99 of the column.
- Rule: ROC-AUC gain > baseline seed SD in all three folds. Secondary: recall at 5% FPR, PR-AUC, cyberattack recall.
- Sensitivity: phy_att_2 and phy_att_3 only (phy_att_1 has about 189 network rows).
