# Agentic preprocessing for fused physical and network data (WDT water testbed)

Capstone experiment: can an LLM agent (Qwen3-32B) design a preprocessing plan that helps different models detect faults and cyberattacks when physical sensor data and network traffic are fused?

**Author / course: Sadeem Alshashaee
## Question
The preprocessing plan must be **model-agnostic**: one plan that is tested on **XGBoost**, an **LSTM** and **Qwen3-32B** itself, and compared against a **no-preprocessing XGBoost baseline**.

## Task
- Three classes: **normal**, **physical fault**, **cyberattack**.
- Physical and network data are fused by **early fusion**: for each second, the physical sensor values and that second's network summary are placed side by side in one input row.
- Evaluation is **leave-one-acquisition-out** on `phy_att_1/2/3`, with `phy_norm` always in training. `phy_att_4` was sealed and used **once** for a final test.

## Data
The WDT dataset is not included in this repository because of its size. It is available on request. To run the notebook, set `PROJECT_DIR` in the setup cell to the folder that holds the physical and network data.

## Method in brief
1. **Network summary:** the agent designed 14 per-second features from the raw packets: packet count, packet size mean/std/max, share of empty responses, distinct ports and addresses, and the shares of the main Modbus function codes.
2. **Fusion plan (Step 20):** the agent proposed a second round of steps from a fixed menu (nearest-second alignment, short gap filling, ratio to a recent baseline, function entropy, dropping two features, and robust scaling for the LSTM). A human reviewed the proposal and corrected factual errors before it was frozen.
3. **Frozen before results:** every plan, rule and protocol was written to a file before the results it governs were seen (see `Records/`).
4. **Keep rule:** a step was kept only if its ROC-AUC gain (attack vs normal) exceeded the baseline's seed-to-seed spread in **all three** folds.
5. **Final test:** the main arms were fixed in advance, run once, and scored with block-bootstrap confidence intervals. No retuning and no rerun.

## Results
**Development folds (average over the three held-out acquisitions, ROC-AUC attack vs normal):**
physical only 0.63, plus a minimal network baseline 0.73, plus the agent's 14 features 0.78.

**Final test on attack_4 (614 covered seconds: 47 attack, 567 normal; "always normal" accuracy = 0.923):**

| Model | Preprocessing / input | ROC-AUC | Recall | Specificity | Precision | F1 | Accuracy | Macro-F1 |
|---|---|---|---|---|---|---|---|---|
| XGBoost | No preprocessing: physical only | 0.600 | 0.043 | 0.918 | 0.041 | 0.042 | 0.851 | 0.481 |
| XGBoost | Minimal network (2 counts, no agent) | 0.900 | 0.936 | 0.874 | 0.381 | 0.541 | 0.878 | 0.736 |
| XGBoost | **Agent's 14 network features (main)** | **0.920** | **1.000** | **0.567** | **0.161** | **0.277** | **0.600** | **0.500** |
| XGBoost | Agent's full fusion plan | 0.944 | 0.801 | 0.934 | 0.502 | 0.617 | 0.924 | 0.787 |
| LSTM | No preprocessing: physical only | 0.328 | 0.043 | 0.833 | 0.021 | 0.028 | 0.771 | 0.453 |
| LSTM | Minimal network (2 counts, no agent) | 0.506 | 0.043 | 0.835 | 0.021 | 0.028 | 0.774 | 0.454 |
| LSTM | **Agent's 14 network features (main)** | **0.858** | **0.801** | **0.831** | **0.287** | **0.422** | **0.829** | **0.673** |
| LSTM | Agent's full fusion plan (robust scaling) | 0.930 | 0.901 | 0.888 | 0.408 | 0.562 | 0.889 | 0.798 |
| Qwen3-32B* | No preprocessing: physical only | 0.427 | 0.000 | 0.867 | 0.000 | 0.000 | 0.533* | 0.348 |
| Qwen3-32B* | Minimal network (2 counts, no agent) | 0.458 | 0.489 | 0.427 | 0.348 | 0.407 | 0.451* | 0.448 |
| Qwen3-32B* | Physical + network features | 0.866 | 1.000 | 0.133 | 0.420 | 0.591 | 0.467* | 0.389 |

\*Qwen was scored on a smaller balanced sample (122 rows), so its accuracy, precision, F1 and macro-F1 are not comparable with the other two models.

**Findings**
- Network data gives most of the gain. On attack_4, ROC-AUC rose from 0.60 to 0.92 (XGBoost) and from 0.33 to 0.86 (LSTM) when the agent's 14 network features were added.
- The agent's 14 features add only a small extra over a minimal 2-count network baseline (XGBoost +0.056, 95% interval 0.001 to 0.206; not significant for the LSTM).
- None of the agent's further fusion refinements passed the keep rule on the development folds. The full plan looked best on attack_4, but this is one test and is reported as a signal, not a confirmed gain.
- At each model's own cutoff, accuracy for the main arms is below the "always normal" score. Cutoffs did not transfer well between recording sessions.

## Limitations
- **One test acquisition.** Its covered rows are almost all DoS (about 42 DoS, 5 scan), so physical-fault and MITM detection could not be tested on attack_4.
- **Truncated network captures.** Only about 36% of attack_4's seconds, and a small part of `phy_att_1`, have network data. Network arms are scored only on covered seconds.
- **Session fingerprinting and shift** between recordings limit absolute detection.
- The LSTM was **not tuned** and is unstable between seeds.
- Qwen was evaluated on a balanced subsample, and the full fusion plan was not tested on Qwen.
- The alignment step looks up to 2 seconds ahead, so it is not usable for real-time detection.
- Late (decision-level) fusion was not tested.

## Decisions made after seeing data (disclosed)
- A date-parsing bug (day-first dates) was found and fixed.
- Attack counts for `phy_att_4` were exposed in Step 11 and removed from later steps.
- The `phy_att_1` sensitivity run, threshold calibration (which did not transfer across sessions), the robust-scaling observation for the LSTM, and the minimal network baseline (Step 10b) were added after earlier results. They are marked as post-hoc.
- Qwen's proposals contained factual errors that were found by human review, not by the automatic checker. They are recorded in `Records/step21_frozen_plan.md`.

## Repository contents
- `WDT_Agentic_Multiclass.ipynb`: the full experiment, with saved outputs. The final test cell (Step 23b) was run once on `phy_att_4`; please do not rerun it as a fresh test.
- `Records/`: frozen plans and protocol written before results (`step14`, `step16`, `step17`, `step21`, `step22`, `step23`), the human adjudication of Qwen's proposal (`step21_adjudication_final.csv`), final-test summaries (`step23_summary.csv`, `step23_pairwise.csv`), the attack_4 coverage check, and the results table image.

## Running it
- Python 3.12 with `xgboost`, `torch`, `huggingface_hub`, `pandas`, `numpy`, `scikit-learn`.
- Create a `.env` file with `HF_TOKEN=your_token_here` for the Qwen steps (never commit this file).
- Set `PROJECT_DIR` to your data folder and run the cells in order.
