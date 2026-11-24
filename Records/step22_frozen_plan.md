# Step 22 frozen LSTM fusion test
Written 2026-11-23T14:13:35 BEFORE any result.
- Model and training: exactly Step 16 (1-layer LSTM, hidden 64, 15 epochs, 10-row sequences, class-weighted loss).
- Paired baseline: Step 16 net_scaled_cc (physical + 14 raw network columns, min-max scaled).
- Steps tested: ratio to 60 s baseline, function entropy, drop port columns, robust scaling of the network columns (median/IQR from training NORMAL rows), and all four together (plan_lstm).
- NOT tested here: alignment tolerance and forward-fill. The LSTM only accepts sequences whose 10 rows all have network data, so these steps only change which rows are eligible and cannot be paired; they were tested in Step 21 (XGBoost).
- Same sequences for every variant: a sequence is eligible if all 10 rows have network data AND finite ratio and entropy values.
- Rule: ROC-AUC gain > baseline seed SD in all three folds. Sensitivity: phy_att_2 and phy_att_3 only.
