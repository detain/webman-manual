# مكون ENV vlucas/phpdotenv

## الوصف
`vlucas/phpdotenv` مكون تحميل متغيرات البيئة، يُستخدم للتمييز بين إعدادات البيئات المختلفة (مثل بيئة التطوير وبيئة الاختبار وغيرها).

## عنوان المشروع

https://github.com/vlucas/phpdotenv
  
## التثبيت
 
```php
composer require vlucas/phpdotenv
 ```
  
## الاستخدام

### إنشاء ملف `.env` جديد في المجلد الجذر للمشروع
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### تعديل ملف الإعدادات
**config/database.php**
```php
return [
    // قاعدة البيانات الافتراضية
    'default' => 'mysql',

    // إعدادات قواعد البيانات المختلفة
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **تنبيه**
> يُنصح بإضافة ملف `.env` إلى قائمة `.gitignore` لتجنب رفعه إلى مستودع الكود. أضف ملف إعدادات نموذجي `.env.example` في المستودع، وعند نشر المشروع انسخ `.env.example` كـ `.env` وعدّل الإعدادات وفق البيئة الحالية. بهذا يحمّل المشروع إعدادات مختلفة حسب كل بيئة.

> **ملاحظة**
> قد تحتوي `vlucas/phpdotenv` على أخطاء في إصدار PHP TS (Thread Safe). يُرجى استخدام إصدار NTS (Non-Thread-Safe). يمكن التحقق من إصدار PHP الحالي بتنفيذ الأمر `php -v`.

## المزيد

زيارة https://github.com/vlucas/phpdotenv
  
