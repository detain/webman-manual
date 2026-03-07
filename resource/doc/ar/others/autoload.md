# التحميل التلقائي

## تحميل ملفات متوافقة مع PSR-0 عبر Composer
يتبع webman مواصفات التحميل التلقائي `PSR-4`. إذا كان مشروعك يحتاج إلى تحميل مكتبات متوافقة مع PSR-0، اتبع الخطوات التالية:

- أنشئ مجلداً `extend` لتخزين مكتبات PSR-0
- عدّل ملف `composer.json` وأضف التالي ضمن `autoload`:

```json
"psr-0" : {
    "": "extend/"
}
```
ستكون النتيجة النهائية مشابهة لما يلي:
![](../../assets/img/psr0.png)

- نفّذ الأمر `composer dumpautoload`
- نفّذ الأمر `php start.php restart` لإعادة تشغيل webman (ملاحظة: إعادة التشغيل الكاملة مطلوبة لتفعيل التغييرات)

## تحميل ملفات محددة عبر Composer

- عدّل ملف `composer.json` وأضف الملفات المطلوب تحميلها ضمن `autoload.files`:
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- نفّذ الأمر `composer dumpautoload`
- نفّذ الأمر `php start.php restart` لإعادة تشغيل webman (ملاحظة: إعادة التشغيل الكاملة مطلوبة لتفعيل التغييرات)

> **ملاحظة**
> الملفات المعرّفة في `autoload.files` داخل composer.json تُحمّل قبل بدء webman. أمّا الملفات المحمّلة عبر `config/autoload.php` في الإطار، فتُحمّل بعد بدء webman.
> التغييرات في ملفات `autoload.files` داخل composer.json تتطلب إعادة التشغيل (restart) لتفعيلها؛ التحديث (reload) لا يكفي. أمّا الملفات المحمّلة عبر `config/autoload.php` فتدعم التحديث السريع (hot-reload)، والتغييرات تُفعّل بعد التحديث (reload).

## تحميل ملفات محددة عبر الإطار
قد لا تتوافق بعض الملفات مع مواصفة PSR ولا تُحمّل تلقائياً. يمكنك تحميلها عبر إعداد `config/autoload.php`، مثلاً:
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **ملاحظة**
 > تُعدّ في `autoload.php` تحميل الملفين `support/Request.php` و`support/Response.php` لأن الملفات نفسها موجودة في `vendor/workerman/webman-framework/src/support/`. عبر `autoload.php` يتمّ إعطاء أولوية لنسخ المجلد الجذر للمشروع، مما يسمح بتخصيص هذين الملفين دون تعديل ملفات مجلد `vendor`. إن لم تحتج لتخصيصهما، يمكنك حذف هاتين الإعدادتين.
