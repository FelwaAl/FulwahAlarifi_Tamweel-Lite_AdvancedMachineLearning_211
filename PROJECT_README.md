# Tamweel Lite | مشروعك النهائي

## الملخص التنفيذي
قارنت ثلاثة نماذج منفردة وثلاث طرق تجميع، واخترت الانحدار اللوجستي لأنه حقق أعلى AP (0.392) ولم يتجاوزه أي تجميع بفارق يفوق الانحراف بين الطيات (0.030). العتبة 0.169 تلتقط 46.9% من حالات التعثر ضمن سعة 12% في كل فترة. لم تحسّن معايرة sigmoid الاحتمالات لأن النموذج كان معايرًا أصلًا. في دفعة التحدي تجاوزت 330 حالة العتبة فاحتُفظ بأعلى 300 فقط. تحتاج فجوة المنطقة الغربية إلى مراجعة، والنتائج تعليمية على بيانات اصطناعية.

## Executive summary
I compared three single models and three ensembles and kept Logistic Regression: it had the best AP (0.392) and no ensemble beat it by more than the fold SD (0.030). The threshold 0.169 catches 46.9% of defaulters within 12% capacity in every period. Sigmoid calibration did not improve the probabilities, as the model was already well calibrated. On the challenge batch, 330 requests passed the threshold and the cap kept the top 300. The western-region gap needs review. All results are educational, on synthetic data.

Decision: KEEP SINGLE / Logistic. Full-batch flags: 300/2500.

اقرأ reports/MODEL_CARD.md والسياسة في artifacts/final_policy.json. الحزمة للتدريب؛ ليست نتيجة تقييم نهائية أو إثبات تسليم. ادمج أدلة أيامك السابقة واحفظ الدفتر المنفذ والعرض.
