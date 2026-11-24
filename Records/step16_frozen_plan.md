# Step 16 frozen LSTM test design
Written 2026-11-21T16:13:26 BEFORE any LSTM result was computed.
- Model: 1-layer LSTM (hidden 64) + linear head, class-weighted cross-entropy (weights from the training fold),
  15 epochs, batch 256, Adam lr 0.001, seeds [0, 1, 2]. Same folds as Step 14b; phy_norm always in training.
- Sample = 10 consecutive rows of one acquisition, label = class of the last row. Sequences never cross acquisitions.
- Variants: raw (unscaled physical), scaled, cleaned_scaled (all rows with 10 rows of history);
  phys_scaled_cc and net_scaled_cc (only samples whose 10 rows all have network data; no availability flag).
- Components: scaling = scaled vs raw; cleaning = cleaned_scaled vs scaled; network = net_scaled_cc vs phys_scaled_cc.
- Rule (unchanged): KEEP only if ROC-AUC attack-vs-normal improves in ALL three folds by more than the baseline seed SD.
- Windowing is part of how an LSTM reads data, so it is not tested as a component here.
