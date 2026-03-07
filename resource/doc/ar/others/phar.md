# حزم Phar

Phar هو نوع من ملفات التغليف في PHP مشابه لـ JAR. يمكنك استخدام Phar لتغليف مشروع webman الخاص بك في ملف Phar واحد لتسهيل النشر.

**شكراً جزيلاً لـ [fuzqing](https://github.com/fuzqing) على طلب الدمج.**

> **ملاحظة**
> تحتاج إلى تعطيل خيار تكوين phar في `php.ini`، وذلك بتعيين `phar.readonly = 0`.

## تثبيت أداة سطر الأوامر
`composer require webman/console`

## التغليف
نفّذ الأمر `php webman build:phar` في الدليل الجذري لمشروع webman. سيتم إنشاء ملف `webman.phar` في دليل `build`.

> إعدادات التغليف في `config/plugin/webman/console/app.php`.

## أوامر التشغيل والإيقاف
**التشغيل**
`php webman.phar start` أو `php webman.phar start -d`

**الإيقاف**
`php webman.phar stop`

**عرض الحالة**
`php webman.phar status`

**عرض حالة الاتصال**
`php webman.phar connections`

**إعادة التشغيل**
`php webman.phar restart` أو `php webman.phar restart -d`

## ملاحظات
* المشاريع المغلفة لا تدعم reload؛ تحتاج إلى إعادة التشغيل لتحديث الكود.

* لتجنب حجم الحزمة الزائد واستهلاك الذاكرة، يمكنك تعيين خيارات `exclude_pattern` و `exclude_files` في `config/plugin/webman/console/app.php` لاستبعاد الملفات غير الضرورية.

* تشغيل webman.phar سينشئ دليل `runtime` في نفس دليل webman.phar، ويستخدم لتخزين الملفات المؤقتة مثل السجلات.

* إذا كان مشروعك يستخدم ملف .env، فضع ملف .env في نفس دليل webman.phar.

* لا تخزن أبداً ملفات رفع المستخدمين داخل حزمة Phar، لأن التعامل مع ملفات المستخدمين عبر بروتوكول `phar://` خطر جداً (ثغرة إلغاء التسلسل Phar). يجب تخزين ملفات المستخدمين بشكل منفصل على القرص خارج حزمة Phar. انظر أدناه.

* إذا كان عملك بحاجة إلى رفع الملفات إلى دليل public، فأنت بحاجة إلى فصل دليل public ووضعه في نفس دليل webman.phar. في هذه الحالة، تحتاج إلى تكوين `config/app.php`.
```
'public_path' => base_path(false) . DIRECTORY_SEPARATOR . 'public',
```
يمكن استخدام الدالة المساعدة `public_path($المسار_النسبي)` للعثور على الموقع الفعلي لدليل public.

* webman.phar لا يدعم العمليات المخصصة على Windows.
