# Step 17 frozen Qwen test design
Written 2026-11-22T16:18:57 BEFORE any Qwen result was computed.
- Model: Qwen/Qwen3-32B, thinking off (/no_think), temperature 0, JSON reply {"class", "attack_score" 0-100}.
- Unit: ONE second (one row), no window. Same sampled rows for every variant: network-covered rows only,
  up to 75 attack and 75 normal rows per fold, random seed 0.
- Variants: Q_raw = task text + raw physical values (no preprocessing);
  Q_ref = Q_raw + summary of normal operation from the training folds + list of values outside the normal range;
  Q_ref_net = Q_ref + the per-second network features (values, normal summary, outside-range list).
- Components: reference = Q_ref vs Q_raw; network = Q_ref_net vs Q_ref.
- Primary metric: ROC-AUC of attack_score, attack vs normal. Unparsable replies get score 50 and are counted.
- Rule: KEEP only if the ROC-AUC gain is positive and larger than the baseline's bootstrap SD (300 resamples
  of test rows) in ALL three folds. (One deterministic pass per row, so there is no seed SD; the bootstrap measures row sampling noise.)
- The normal summary and the outside-range lists are an adapter built by the researchers; Qwen does not choose them.
