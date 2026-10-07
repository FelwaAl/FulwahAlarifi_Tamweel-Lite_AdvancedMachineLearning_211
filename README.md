# FulwahAlarifi_Tamweel-Lite_AdvancedMachineLearning_211
Final project of SADAIA's program (Advanced Machine Learning) class 211.
 
An end-to-end tabular machine-learning project that predicts a **synthetic financing default within 90 days of application**, validates honestly across time and customers, selects a cost-sensitive decision threshold under a **12% review capacity**, checks calibration and interpretability, and delivers a reproducible final batch policy.
 
**Course:** SDA-DSC-211 — Advanced Machine Learning Methods (SDAIA Academy) · **Project type:** Individual five-day project
 
> **Educational use only.** All data is fully synthetic. A flag (`decision = 1`) means *refer for review* in a simulation. It is not an approval, a refusal or a statement about any real person or region.
 
---
 
## Final decision at a glance
 
| Item | Result |
|---|---|
| Final model | **Logistic Regression** (KEEP SINGLE) |
| Mean OOF Average Precision | **0.392** (fold SD 0.030) |
| Decision rule | `probability >= 0.1223` (calibrated scale; raw OOF threshold 0.1689) |
| OOF recall / precision at threshold | **46.9%** / 34.3% (84 of 179 defaults caught) |
| Busiest-period flag rate (OOF) | 11.7% — within the 12% cap in all three periods |
| Challenge batch | 2,500 applications → 330 above threshold → **300 flagged** (cap applied) |
| Challenge performance | **Not claimed** — challenge labels are unavailable |
 
![Ensemble comparison](artifacts/day5_ensemble_comparison.png)
 
---
 
## The problem
 
A lender must decide which new applications a limited review team should examine. Missing a future default (false negative) is treated as **10× as costly** as reviewing a customer who would have repaid (false positive), and the team can review at most **12% of applications in each period**.
 
- **Target:** `default_within_90d = 1` — a synthetic default within 90 days *after* the application (not "90 days past due").
- **Training data:** 10,000 applications (2022–2024), 7.89% positive.
- **Challenge data:** 2,500 later applications (2025) from separate customers, without labels.
- **Educational loss:** `10 × FN + 1 × FP`, in teaching units only.
---
 
## Five-day build
 
| Day | Focus | Key evidence |
|---:|---|---|
| 1 | Baseline vs. gradient boosting | `day1_model_comparison.csv`, ROC/PR and learning curves |
| 2 | Leakage audit, time- and customer-aware validation, bounded Optuna search | `leakage_audit.csv`, `fold_audit.csv`, `validation_summary.csv` |
| 3 | Class imbalance, threshold sweep, capacity and regional audit | `DECISION_CARD.md`, `threshold_metrics.json` |
| 4 | SHAP, permutation importance, calibration and stability | `INTERPRETABILITY_REPORT.md` |
| 5 | Worth-It Gate for ensembles, final model, batch policy and delivery | `ENSEMBLE_DECISION.md`, `MODEL_CARD.md`, `submission.csv` |
 
---
 
## Key results
 
### Day 1 — Fair model comparison
 
On the same 2,000-row comparison split, the three models were close in ranking quality:
 
| Model | ROC-AUC | AP | Train time |
|---|---:|---:|---:|
| Logistic Regression | 0.821 | 0.326 | 0.06 s |
| XGBoost | 0.812 | 0.334 | 0.45 s |
| LightGBM | 0.814 | 0.325 | 0.29 s |
 
Logistic Regression matched the boosters at a fraction of the cost, so it was the Day 1 candidate. Day 5 later confirmed this choice under stricter validation.
 
![Day 1 ROC and PR curves](artifacts/day1_roc_pr.png)
 
### Day 2 — Honest validation and leakage control
 
Two columns (`days_past_due_60`, `collection_calls`) record information *after* the decision date. Including them produced an almost perfect, and meaningless, score:
 
| Validation scheme | Mean AP |
|---|---:|
| Leaky random split (control) | 0.999 |
| Clean random split | 0.311 |
| Clean time + customer split, fixed parameters | **0.315** |
| Clean time + customer split, tuned (Optuna) | 0.313 |
 
The honest scheme keeps customers separate, requires every training label to mature for 90 days before the validation period starts, and fits imputers inside each fold. The bounded Optuna search did not beat the fixed configuration, so the simpler setup was kept.
 
![Validation comparison](artifacts/day2_validation_comparison.png)
 
### Day 3 — Cost-sensitive threshold under capacity
 
| Threshold | Loss units | Flag rate | Feasible under 12%? |
|---|---:|---:|---|
| Default 0.5 | 2,403 | 19.9% | No |
| Unconstrained minimum loss (0.449) | 2,275 | 22.6% | No |
| **Chosen, minimum loss within capacity (0.658)** | **2,639** | 10.4% | **Yes** |
 
The chosen threshold costs 236 more units than 0.5. That is the price of respecting capacity, not a saving. The threshold stayed the same when the false-negative cost was varied (8, 10, 12), because capacity, not cost, determines it.
 
![Capacity and regions](artifacts/day3_capacity_regions.png)
 
### Day 4 — Interpretation and calibration
 
SHAP and permutation importance explained the **Day 4 weighted LightGBM** in log-odds units, and calibration was measured on a separate period. Full evidence is in [`INTERPRETABILITY_REPORT.md`](INTERPRETABILITY_REPORT.md).
 
These explanations **do not transfer automatically** to the final Logistic Regression. The model card lists rechecking feature contributions on the final model as follow-up work.
 
### Day 5 — Ensemble Worth-It Gate
 
Three single models and three ensembles were compared with nested forward OOF predictions (2,155 rows, folds 2023Q1, 2023Q3, 2024Q1). An ensemble had to beat the best single model by more than one fold SD without worsening Brier or ECE.
 
| Candidate | Mean AP | Fold SD | Brier | ECE | Passes gate |
|---|---:|---:|---:|---:|---|
| **Logistic** | **0.392** | 0.030 | **0.0633** | 0.019 | Reference |
| Weighted ensemble | 0.389 | 0.029 | 0.0633 | 0.018 | No |
| Stacking | 0.383 | 0.029 | 0.0660 | 0.031 | No |
| Equal average | 0.372 | 0.033 | 0.0643 | 0.020 | No |
| XGBoost | 0.353 | 0.029 | 0.0657 | 0.023 | No |
| LightGBM | 0.345 | 0.043 | 0.0661 | 0.023 | No |
 
The base models were highly correlated (0.88–0.96), so combining them added little. **Decision: KEEP SINGLE.**
 
![Model diversity](artifacts/day5_diversity.png)
 
### Calibration
 
A sigmoid mapping was fitted on a reserved calibration period (836 rows, 78 defaults). It **did not improve** the probabilities:
 
| Metric | Raw | After sigmoid |
|---|---:|---:|
| Brier | 0.0765 | 0.0781 |
| ECE | 0.021 | 0.035 |
| Log-loss | 0.266 | 0.277 |
| AP / ROC-AUC | 0.288 / 0.789 | unchanged |
 
The mapping preserves order, so the same applications are flagged either way; only the reported probability values change. These are fit diagnostics on the rows used to learn the sigmoid, not an independent evaluation.
 
![Calibration fit](artifacts/day5_calibration_fit.png)
 
### Batch policy on the challenge set
 
The threshold is applied first. Then, if more than 12% pass, only the highest-probability applications up to the cap are kept, and tied scores stay together.
 
![Challenge capacity](artifacts/day5_challenge_capacity.png)
 
### Regional audit (descriptive)
 
| Region | False-positive rate | Recall |
|---|---:|---:|
| Eastern | 6.3% | 45.7% |
| Central | 8.0% | 50.0% |
| Other | 8.1% | 46.9% |
| Western | **10.3%** | **44.7%** |
 
The western region had the highest false-alarm rate and lowest recall, the same pattern as Day 3. With 35–49 defaults per region this is noisy, and it is not a fairness certificate, but it needs review.
 
![Policy by region](artifacts/day5_policy_regions.png)
 
---
 
## Interpretation and limitations
 
- Development evidence comes from out-of-fold predictions on data used throughout the course. It is **not an untouched final test**.
- The threshold was chosen on the same OOF labels used to report its loss, so the loss (1,111 units) is **optimistic**.
- Fold SD comes from three overlapping forward folds. It is **descriptive**, not a confidence interval.
- Calibration slightly worsened the probabilities, so the probability values themselves should be monitored.
- Before capping, the challenge batch exceeded capacity (13.2%), which suggests score drift or a higher-risk batch.
- Day 4 explanations belong to a different model than the final one.
- The model must not be used for real financing decisions or for conclusions about real people or regions.
## Monitoring plan
 
- Track each batch's pre-cap flag rate against 12%.
- Track score distribution and prevalence for drift.
- After 90-day outcomes mature, recheck AP, Brier and ECE, and whether the sigmoid should be removed through a governed update.
- Recheck regional false-positive rates and recall with their denominators, especially for the western region.
- Develop and validate any change to the model or threshold on new data, never on the batch being judged.
---
 
## Repository structure
 
```
├── README.md
├── MODEL_CARD.md                 # final model card
├── DECISION_CARD.md              # Day 3 threshold decision
├── INTERPRETABILITY_REPORT.md    # Day 4 explanation and calibration
├── ENSEMBLE_DECISION.md          # Day 5 Worth-It Gate
├── submission.csv                # application_id, probability, decision
├── metrics.json                  # final metrics and batch audit
├── notebooks/                    # executed notebooks 00, 01–05, 99
├── artifacts/                    # CSV, JSON and figures from every day
│   ├── final_model/              # exported final model
│   └── final_policy.json         # frozen threshold, mapping and batch rule
├── reports/                      # report copies generated by the labs
├── submission/                   # submission file and manifest
├── scripts/                      # course pipeline, inference and replay
├── data/                         # synthetic course data and data contract
└── presentation/                 # five-slide final presentation (PDF)
```
 
## Reproduce
 
Each notebook in `notebooks/` runs on free Google Colab CPU and contains its saved outputs.
 
To check that the saved submission is reproduced exactly from the exported model, without retraining:
 
```bash
pip install -r requirements-colab.txt -c constraints.txt
cd scripts
python replay_final.py      # prints REPLAY_MATCH on success
python rebuild_final.py     # full retrain from data; prints REBUILD_MATCH
```
 
## Data
 
The Tamweel Lite data is fully synthetic and was created for the course. It contains no real customers and no real regional or demographic statistics. Monetary values are simulated riyals, and decision losses are educational units.
 
---
 
## الملخص التنفيذي
 
قارنت ثلاثة نماذج منفردة وثلاث طرق تجميع، واخترت الانحدار اللوجستي لأنه حقق أعلى AP (0.392) ولم يتجاوزه أي تجميع بفارق يفوق الانحراف بين الطيات (0.030). العتبة تلتقط 46.9% من حالات التعثر ضمن سعة 12% في كل فترة. لم تحسّن معايرة sigmoid الاحتمالات. في دفعة التحدي تجاوزت 330 حالة العتبة فاحتُفظ بأعلى 300 فقط. تحتاج فجوة المنطقة الغربية إلى مراجعة، والنتائج تعليمية على بيانات اصطناعية.
 
---
## Training-program attribution
This project was completed for the **Advanced Machine Learning Methods (SDA-DSC-211)** project, delivered by **SDAIA Academy via Learning Space** as a five-day, on-site, 20-hour program. Session: **October 2026**.
 
Training-program reference: [SDAIA Academy on GitHub](https://github.com/SDAIAAcademy).
 
