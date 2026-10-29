# نظام النسيم — نسخة تحويل PostgreSQL لـ Vercel + Supabase

> **الحالة الحالية: مرشح برمجي للتحويل، وليس إصدار إنتاج مُختبرًا بعد.** اجتازت الملفات فحوصات JavaScript syntax، لكن لا يوجد اتصال تكاملي مُختبر مع قاعدة Supabase بعد.

## ما الذي تغير؟
- `server.js`: خادم Express يستخدم `pg` للوصول إلى PostgreSQL في Supabase، مع واجهة النظام الحالية و120 مسار API.
- `api/index.js` و`vercel.json`: مدخل نشر Vercel مع تضمين ملفات الواجهة وإعادة توجيه الطلبات إلى Express.
- `supabase/nasim-schema-migrations/`: مخطط مستقل اسمه `nasim` يحوي 51 جدولًا، وملف بيانات مصدره نسخة SQLite المرفقة (449 سجلًا)، وقيود العلاقات، وإعدادات sequences، وStorage bucket للمرفقات.
- مرفقات الحجوزات تُحفظ في Supabase Storage بدل القرص المؤقت في Vercel.
- `legacy/`: ملفات مرجعية من نسخة SQLite المحلية فقط، لا يستخدمها الخادم الجديد.

## إعداد قاعدة Supabase
مشروع Supabase الحالي قد يحتوي على جداول في `public`؛ لذلك لا تشغّل ملفات `supabase/migrations` العامة. استخدم الملفات المعزولة التالية من مجلد `supabase/nasim-schema-migrations` بالترتيب، من SQL Editor، ملفًا في كل مرة:

1. `0001_schema.sql` — إنشاء الجداول في `nasim`.
2. `0002_data.sql` — تحميل بيانات نسخة SQLite التي تم تصديرها مسبقًا (449 سجلًا).
3. `0003_constraints_security.sql` — إضافة قيود العلاقات بعد إدخال البيانات.
4. `0004_reset_identity_sequences.sql` — مزامنة أرقام identity مع أعلى ID موجود.
5. `0005_storage_bucket.sql` — إنشاء bucket خاص لمرفقات الحجوزات.

**لا تشغّل أداة `scripts/import-sqlite-to-supabase.js` بعد تطبيق `0002_data.sql`**؛ الملف الثاني يحتوي بالفعل على بيانات النسخة الاحتياطية. الأداة بديل لنقل البيانات يدويًا إذا اخترت عدم تطبيق `0002_data.sql`.

## متغيرات البيئة في Vercel
انسخ أسماء المتغيرات من `.env.example` وأدخل القيم الحقيقية في Vercel Project Settings → Environment Variables:

- `SUPABASE_DB_URL`: اتصال PostgreSQL، ويُفضّل Supabase pooler المناسب للبيئة.
- `JWT_SECRET`: سر طويل وعشوائي للجلسات.
- `SUPABASE_URL`: رابط مشروع Supabase.
- `SUPABASE_SERVICE_ROLE_KEY`: مفتاح خادم سري للوصول إلى Storage؛ لا يوضع في الواجهة أو GitHub.
- `SUPABASE_STORAGE_BUCKET`: `nasim-booking-files`.
- `PG_POOL_MAX`: ابدأ بقيمة `5`.

## فحص الملفات
من مجلد المشروع شغّل:

```bash
npm run check
```

هذا الفحص يراجع بناء JavaScript فقط؛ **لا يثبت** أن كل استعلام SQL يعمل على PostgreSQL. يلزم اختبار Preview حقيقي بعد إعداد Supabase.

## قبل اعتبار النسخة جاهزة
اختبر تسجيل الدخول والصلاحيات، إنشاء/تعديل/البحث عن الحجز، التعارضات، المدفوعات والإيصالات، الخصومات، التقارير، أوامر الشغل، المخزون، مرفقات الحجوزات، ثم راجع عدد السجلات قبل وبعد النقل. لا تنقل هذه النسخة إلى الإنتاج قبل نجاح الاختبارات.

## حدود معروفة حاليًا
- لم يتم تطبيق migrations على مشروع Supabase أو نقل البيانات فعليًا من داخل هذه الحزمة.
- لم تُجرَ اختبارات تكامل API مع قاعدة PostgreSQL حقيقية.
- تنزيل واستعادة ملفات SQLite القديمة غير مدعومين في نسخة PostgreSQL؛ مساراتهما ترجع رسالة توضيحية بدل محاولة التعامل مع قاعدة PostgreSQL كملف SQLite.
- يلزم وضع مفاتيح البيئة في Vercel يدويًا؛ لا توجد مفاتيح حقيقية مضمنة في المشروع.
