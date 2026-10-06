# بطاقة قرارك — Tamweel Lite

**الحالة:** جاهزة للمراجعة؛ لا تعني اعتمادًا أو درجة
**مصدر الأرقام:** LIVE · **الاستراتيجية:** weighted · **الصفوف:** 5,039 OOF

**المهمة:** الفئة الموجبة `default_within_90d=1` تعني حدث تعثر اصطناعي خلال90 يومًا بعد الطلب. كل طية تحقق طلبات لاحقة، وتستبعد عملاءها من التدريب وتشترط نضج نتيجة التدريب قبل بدايتها. المعرّفات والتاريخ خارج المدخلات.

| الدليل | القيمة |
|---|---:|
| العتبة المقيدة، بالقيمة الكاملة | 0.6583471436014694 |
| عتبة أقل خسارة دون قيد | 0.44863935722081005 |
| Recall | 40.89% |
| Precision | 29.85% |
| AP مجمع منOOF | 0.3100 |
| الإشارات | 526 من 5,039 |
| FN / FP | 227 / 369 |
| الخسارة التعليمية | 2639 وحدة |
| خسارة0.5 | 2403 وحدة؛ ضمن السعة: False |
| التغير عن0.5 | +236 وحدة؛ الموجب زيادة |
| خسارة لكل10,000 طلب، تطبيع حسابي | 5237.15 وحدة |
| فجوة معدل الإنذار الخاطئ بين المناطق | 0.648 نقطة مئوية |

**القاعدة:** درجة ≥ 0.6583471436014694 تعني إشارة مراجعة داخل التمرين؛ غير ذلك بلا إشارة. لا تتخذ موافقة أو رفض تمويل حقيقي. احفظ الدقة الكاملة؛ تقريب العتبة قد يغيّر حجم الطابور.

**السياسة:** FN=10 وFP=1 وحدات تعليمية، وسعة 12% لكل فترة بعد التقريب لأسفل. ليست ريالات فعلية أو رسوم أدوات أو خصمًا من الدرجة.

**دليل السعة:** الفترة 1: 137/195, الفترة 2: 183/200, الفترة 3: 206/207.

## لماذا اخترت هذه العتبة؟
I chose 0.6583471436014694 (flag when score >= threshold) because it has the lowest observed OOF loss among rules that stay within the 12% review capacity in every validation period. The 0.5 rule (2,403 units) and the unconstrained minimum-loss rule at 0.449 (2,275 units) lose less, but they flag 19.9% and 22.6% of requests, overloading reviewers, so their loss cannot actually be achieved. My rule flags 526 requests (10.4% pooled), catching 157 of 384 defaulters (recall 40.9%, precision 29.9%) for 2,639 units. The threshold comes from observed loss, not the 1/11 formula, because class weighting inflated the raw scores and they are not calibrated probabilities. The full threshold value is kept because rounding could change which requests are flagged.

## الخسارة والسعة
Capacity forces a higher threshold than loss alone would choose. Compared with the unconstrained minimum-loss rule (0.449), my rule misses 89 more defaulters (+890 units) but raises 526 fewer false alarms (-526 units), a net +364 units; compared with 0.5 it is +236 units (+65 FN, -414 FP). Because a missed defaulter costs 10 times a false alarm, the extra cost comes almost entirely from FN. This +364 units is the price of the 12% per-period limit. Loss could fall without breaking that limit only by raising review capacity or by improving the model's ranking so more true defaulters fall in the flagged top 12%.

## فرق المناطق وما يحتاج إلى مراجعة
With one shared threshold, the false-positive rate ranges from 7.64% (other) to 8.29% (western), a gap of 0.65 percentage points; every region has over 1,000 non-defaulters, so no low-support warning. The small FPR gap does not prove fairness: recall varies more, from 48.1% (other) to 34.6% (western), a 13.5-point gap, and western is worst on both FPR and recall. Each region has only about 80-104 defaulters, so a few cases shift recall by several points and the gap may be partly noise. This is a descriptive audit, not a significance test or causal finding about geography. The western recall gap needs review over more data, and should not be hidden by assigning separate regional thresholds without a governed policy.

## حدود النتيجة
The threshold was chosen and evaluated on the same OOF development labels, so the 2,639-unit loss is optimistic evidence, not an independent test; future loss is likely somewhat higher. Only 5,039 of 10,000 rows (50.4%) have OOF predictions because the earliest warm-up rows cannot be validated. Class weighting inflated the raw scores, so they are not calibrated probabilities until Day 4 checks. The policy is simplified: loss units are educational, FN = 10 is an assumption, and review cost and review effectiveness are ignored. Capacity feasibility in past periods does not guarantee future capacity, and lower AP in fold 3 hints at drift that would need monitoring.

OOF تغطي 50.39% من التدريب و100% من الصفوف المؤهلة؛ 4,961 صفًا تمهيديًا بلا تنبؤ. اختيار العتبة وتقدير خسارتها هنا يستخدمان أهدافOOF نفسها؛ هذه نتيجة تطوير لا اختبار نهائي. لم نستخدم التحدي. المقارنة الجغرافية وصفية وليست شهادة عدالة، والأوزان لا تضمن معايرة الدرجات.

## سؤالك الأول: لماذا قد تخدعكAccuracy؟
Only about 8% of applications default, so a rule that flags nobody reaches about 92% accuracy while catching zero defaulters (recall = 0). High accuracy is driven by correctly leaving the non-defaulter majority alone. Accuracy also counts every error as 1, but our policy makes a missed defaulter cost 10 times a false alarm. At my chosen threshold, 227 FN and 369 FP are 596 errors to accuracy, but 10x227 + 369 = 2,639 loss units, with 86% of the loss coming from the missed defaulters. So I judge rules by loss, recall, precision and AP together, not accuracy.

## سؤالك الثاني: لماذا تختار علىOOF؟
Predictions on training rows are over-optimistic because the model memorises the rows it was trained on, so a threshold chosen on them would fail on new applications. OOF scores come from models that never saw those rows and were trained only on earlier periods, which mimics real use where the model scores future customers using past data and exposes changes over time. The challenge data stays closed so we keep one honest final check; using it to pick the threshold would leak its labels into our decision. Even OOF selection is slightly optimistic, because the same development labels are used to choose the threshold.

أدلتك في `artifacts/threshold_metrics.json` و`day3_period_capacity.csv` و`day3_region_audit.csv` و`day3_cost_sensitivity.csv` و`cost_curve.png`. الحساسية سيناريوهات ±20% لخسارةFN، وليست فترات ثقة. راجع السعة والمعايرة عند تغير البيانات؛ لا تفترض ثباتهما مستقبلًا.
