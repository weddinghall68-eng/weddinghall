# أداة نقل بيانات SQLite إلى Supabase — نظام النسيم

هذه الأداة مخصصة لنقل البيانات فقط، ولا تحوّل `server.js` تلقائيًا إلى PostgreSQL.

## متطلبات قبل التشغيل
1. نسخة احتياطية إضافية من قاعدة SQLite.
2. تطبيق `0001_schema.sql` ثم `0003_constraints_security.sql` و`0004_reset_identity_sequences.sql` و`0005_storage_bucket.sql` على مخطط `nasim`. لا تطبق `0002_data.sql` إذا كنت ستستخدم الأداة أدناه لنقل البيانات يدويًا.
3. يجب أن تكون جداول `nasim` فارغة. الأداة لا تستخدم `ON CONFLICT` ولا تمسح أو تستبدل أي سجل؛ لو كانت هناك مفاتيح مكررة فستتراجع المعاملة كاملة.
4. السماح بالاتصال بقاعدة البيانات من الجهاز الذي يشغّل الأداة، باستخدام رابط PostgreSQL من إعدادات Supabase. لا تضع الرابط في GitHub ولا ترسله داخل المحادثة.

## تشغيل الأداة من جهازك
من مجلد المشروع:

```bash
cd migration-tools
npm install
```

ثم على Windows PowerShell، عيّن المسار ورابط PostgreSQL في الجلسة الحالية فقط:

```powershell
$env:SQLITE_FILE="C:\path\to\nasim.sqlite"
$env:DATABASE_URL="YOUR_SUPABASE_POSTGRES_CONNECTION_STRING"
node ..\scripts\import-sqlite-to-supabase.js
```

لا تستخدم رابط اتصال قاعدة البيانات كقيمة في ملفات المشروع. لا تنفّذ النقل على جداول `public` أو على قاعدة فيها بيانات أخرى؛ الأداة تستهدف `nasim` فقط.

## تنبيه مهم
هذه الأداة تنقل البيانات فقط. يوجد الآن مرشح Backend PostgreSQL في `server.js` لكنه لم يُختبر تكامليًا مع قاعدة حقيقية؛ اختبره على Preview قبل الإنتاج. لا تنشر هذه النسخة باعتبارها النسخة النهائية.
