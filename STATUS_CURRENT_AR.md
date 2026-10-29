# الحالة الحالية — تحويل النسيم إلى Supabase/Vercel

## تم تنفيذه في ملفات المشروع
- استبدال SQLite runtime في `server.js` بطبقة PostgreSQL async تعتمد على `pg`.
- ترقية عمليات `get/all/run` إلى عمليات awaited، مع دعم معاملات PostgreSQL.
- تجهيز `api/index.js` و`vercel.json` للنشر على Vercel.
- ترحيل مرفقات الحجوزات إلى Supabase Storage.
- تجهيز `.env.example` وملف إنشاء bucket.
- الاحتفاظ بنسخة الخادم المحلي الأصلية في `legacy/` للرجوع إليها.
- فحص نحوي: `node --check server.js`, `node --check api/index.js`, `node --check public/app.js`, و`node --check scripts/import-sqlite-to-supabase.js` نجحت.
- تدقيق الترحيل: 51 جدولًا مصدرًا، و51 جدولًا في مخطط PostgreSQL، و449 سجلًا في ملف البيانات المصدّر.

## ما زال غير مكتمل / لم يتم التحقق منه
- لم يتم تشغيل backend فعليًا على قاعدة Supabase؛ لم تُختبر استعلامات SQL في PostgreSQL حقيقي.
- لم يتم تطبيق ملفات SQL على مشروع Supabase ولا نقل البيانات فعليًا من داخل هذه الجلسة.
- يجب اختبار جميع الوظائف على Vercel Preview؛ نجاح syntax لا يعني نجاح التطبيق.
- نسخ/استعادة SQLite القديمة معطلة في نسخة PostgreSQL، ويلزم اعتماد آلية backup/restore ملائمة لـ Supabase.

## تحذير
لا تعتبر النسخة جاهزة للإنتاج حتى تطبيق مخطط `nasim` على بيئة اختبار، واختبار مسارات API الأساسية والمالية والمرفقات، ومطابقة عدد السجلات.
