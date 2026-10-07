# قرار التجميع

KEEP SINGLE — Logistic

KEEP SINGLE: Logistic Regression. It had the best mean AP across three forward folds (0.392, fold SD 0.030) and the best Brier (0.0633). No ensemble beat it: Weighted 0.389 (-0.002), Stack 0.383 (-0.009), Equal 0.372 (-0.020), so none cleared the gate of gaining more than one fold SD. The base models were highly correlated (0.88-0.96), so combining them had little to add. Stack also had the worst ECE (0.031) because it rescales probabilities. Weighted's 0.002 gap is noise, and three models cost more to run, monitor and explain, so the extra complexity did not earn its place.

Candidates were compared with nested forward OOF on 2,155 rows across folds 2023Q1, 2023Q3 and 2024Q1: training always precedes validation with time for the 90-day outcome to mature, customers are separated, and ensemble weights and the stacker were learned on inner OOF only. Limits: the three folds share overlapping training histories, so fold SD is descriptive, not a confidence interval or significance test. The threshold was chosen on the same OOF labels, so its loss (1,111 units) is optimistic. The training data was used earlier in the course, so this is not a final untouched test, and challenge labels are unavailable.

الدليل: artifacts/ensemble_comparison.csv وday5_ensemble_gate.json. SD وصفي، وليس اختبار دلالة.
