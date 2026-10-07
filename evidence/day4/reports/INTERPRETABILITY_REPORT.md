# تقريرك: التفسير والمعايرة — Tamweel Lite

**الحالة:** جاهز للمراجعة؛ لا يعني اعتمادًا أو درجة

**مصدر التفسير:** LIVE. **النموذج والمعايرة:** LIVE. **السعة:** CAPACITY_REVIEW_REQUIRED.

## النموذج والأدوار
LightGBM موزون، 80 شجرة. الهدف حدث تعثر اصطناعي خلال90 يومًا بعد الطلب. الأدوار منفصلة زمنيًا وبالعملاء: تدريب 2516، معايرة 584 (40 موجب)، سياسة 589، تقييم 1733. الفجوات والتداخلات مستبعدة. سبق استخدام بيانات التقييم في الدورة، فهي ليست اختبارًا نهائيًا لم يمسّ.

## التفسير العام والمحلي
Permutation يقيس انخفاضAP على التقييم؛ إشارات المنطقة تُبدّل معًا. SHAP يفسر النموذج الخام بوحدةlog-odds وخلفية مسارات أشجار التدريب. base+sum(SHAP)=raw margin، ثمsigmoid للمجموع فقط. القيم ليست نقاط احتمال ولا تفسيرًا مباشرًا للنموذج المعاير.

Global explanations show what the model relies on across all applicants: held-out permutation importance and mean |SHAP| agree that bureau_score (AP drop 0.127; mean |SHAP| 0.904) and dti (0.069; 0.544) dominate, followed by loan_amount_sar. Local SHAP explains one applicant: TR-009585 (raw score 0.903, calibrated 0.480) was driven by bureau_score 497 (+2.27 log-odds) and dti 1.28 (+1.20). Global importance is an average, so a reviewer needs the local explanation to know which reason applies to the person in front of them. Training gain differs from held-out importance (months_employed: high gain 2,482, AP drop 0.003), so I trust the held-out measure.

SHAP values explain the raw weighted model in log-odds, not probability points: raw margin = base value + sum of SHAP, and the raw score is sigmoid of that total. Contributions cannot be converted to probability one by one and added, and sigmoid(base value) is not the mean predicted probability. The background is the training tree-path distribution, and displayed values are inputs after fit-only imputation. The additivity error was about 6e-15. SHAP explains the raw model, not the calibrated probability.

الطلب الاصطناعي TR-009585: الدرجة الخام 0.90308 والاحتمال المعاير 0.47952. اختير أعلى درجة داخل عينةSHAP دون استخدام النتيجة الفعلية.
- استخدم النموذج درجة ائتمانية اصطناعية عند الطلب بالقيمة 497 لرفع درجته الخام؛ لا يثبت ذلك السببية. (SHAP=+2.2693 log-odds؛ قيمة معوضة: False)
- استخدم النموذج نسبة الالتزام مع القسط المقترح إلى الدخل بالقيمة 1.2806 لرفع درجته الخام؛ لا يثبت ذلك السببية. (SHAP=+1.1981 log-odds؛ قيمة معوضة: False)
- استخدم النموذج مبلغ التمويل المطلوب بالقيمة 93437.3 لرفع درجته الخام؛ لا يثبت ذلك السببية. (SHAP=+0.2007 log-odds؛ قيمة معوضة: False)

Reason statements come from the largest positive SHAP contributions (at most three) and describe the model's behaviour, not causes of default. For TR-009585 all three inputs were observed, not imputed. Correlated features can share signal, so one feature's contribution may be split with others and look smaller. Changing a feature does not guarantee a different outcome. The bureau_score +/-1 check left the probability (0.4795) and top three reasons unchanged, but that is a narrow check for one applicant, not a guarantee for other applicants, features or periods.

## دليل المعايرة
على 1733 صفًا و139 موجب: Brier 0.113027 → 0.067112؛ ECE 0.146871 → 0.022486. عشر حاويات متساوية العرض مع أعدادها فيday4_reliability_bins.csv. AP 0.258677 → 0.258677؛ ROC-AUC 0.770804 → 0.770804. هذه نتائج هذه العينة وليست ضمانًا لتحسن مستقبلي.

The sigmoid was learned on the separate calibration period (584 rows, 40 defaults) and measured on evaluation (1,733 rows, 139 defaults). Brier fell from 0.1130 to 0.0671, ECE from 0.1469 to 0.0225 and log-loss from 0.358 to 0.247, while ROC-AUC (0.771) and AP (0.259) were unchanged because the mapping keeps the score order. Class weighting had inflated raw scores: TR-009585 went from 0.903 raw to 0.480 calibrated. Reviewers should see the calibrated probability. ECE depends on bin choice, and sparse high-probability bins are weak evidence.

## الاستقرار
200 تكرارbootstrap صالح بسحب العملاء؛ فترات مئينية95% مع تثبيت النموذج والمعاير. لا تشمل تعلم النموذج أو المعايرة أو الانجراف المستقبلي، ولا تصف احتمال فرد. انحرافAP بين ربعي التقييم وصفي فقط. اختبارbureau_score±1 نُفذ؛ راجع day4_local_stability.csv.

A paired customer-cluster bootstrap (200 replicates) gave a 95% range for AP of 0.197 to 0.338 and for Brier change of -0.054 to -0.037. The Brier range is entirely below zero, so the calibration improvement was consistent across resamples. The wide AP range reflects only 139 defaulters. The model and calibrator were fixed, so the range excludes uncertainty from retraining, refitting the calibration and future drift, and it is not an interval for one applicant's probability. Quarterly AP was similar (Q3 0.275, Q4 0.271), which is descriptive only.

## العتبة ومنطقة المراجعة
العتبة الخام 0.5881953696965011 اختيرت علىpolicy بخسارة10×FN+FP وسقف12% ثم نُقلت إلى 0.17331013263107387. لم تعدل باستخدام التقييم. المنطقة[0.15331, 0.19331] تشخيصية بعرض±0.02 وليست فترة ثقة. الاتحاد يحسب الطلب مرة واحدة.
- 2024Q3: السقف 100، الإشارات 97، اتحاد المراجعة 109.
- 2024Q4: السقف 107، الإشارات 109، اتحاد المراجعة 122.

The threshold was chosen on the policy period only (raw 0.588, flagging 68 of 589 within capacity 70) and transported to a calibrated 0.173. On evaluation, Q3 risk flags (97) fit capacity (100), but adding the +/-0.02 review zone raised the review set to 109. In Q4, risk flags alone (109) already exceeded capacity (107), and with the zone the review set reached 122. Status: CAPACITY_REVIEW_REQUIRED. I did not retune the threshold or raise the ceiling using evaluation results; a revised policy must be developed and tested on new data.

عند تجاوز السعة، وثّق الحاجة إلى تصميم سياسة جديدة على بيانات تطوير وتقييمها بدليل جديد. لا ترفع السقف ولا تقص الحالات بعد رؤية النتيجة. التفسير ليس سببية أو شهادة عدالة، والخسارة وحدات تعليمية لا رسوم أو خصم درجات. لا يستخدم هذا التمرين لتمويل حقيقي.
