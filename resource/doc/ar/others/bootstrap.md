# تهيئة الأعمال

في بعض الأحيان نحتاج إلى تنفيذ بعض عمليات تهيئة الأعمال بعد بدء العملية، وتُنفَّذ هذه التهيئة مرة واحدة فقط خلال دورة حياة العملية، مثل إعداد مؤقت بعد بدء العملية أو تهيئة اتصال قاعدة البيانات وغير ذلك. فيما يلي سنشرح هذا الموضوع.

## المبدأ
وفقًا لما ورد في **[سير التنفيذ](process.md)**، يقوم webman بعد بدء العملية بتحميل الفئات المعرفة في `config/bootstrap.php` (بما فيها `config/plugin/*/*/bootstrap.php`) وينفذ طريقة start للفئة. يمكننا إضافة كود الأعمال إلى طريقة start لإكمال تهيئة الأعمال بعد بدء العملية.

## الإجراء
لنفترض أننا نريد إنشاء مؤقت للإبلاغ الدوري عن استهلاك الذاكرة للعملية الحالية، ونسمي هذه الفئة `MemReport`.

#### تنفيذ الأمر

نفّذ الأمر `php webman make:bootstrap MemReport` لإنشاء ملف التهيئة `app/bootstrap/MemReport.php`

> **تلميح**
> إذا لم يكن لديك `webman/console` مثبتًا، نفّذ الأمر `composer require webman/console` للتثبيت

#### تعديل ملف التهيئة
عدّل الملف `app/bootstrap/MemReport.php` بمحتوى شبيه بالتالي:
```php
<?php

namespace app\bootstrap;

use Webman\Bootstrap;

class MemReport implements Bootstrap
{
    public static function start($worker)
    {
        // هل هي بيئة سطر الأوامر؟
        $is_console = !$worker;
        if ($is_console) {
            // إذا كنت لا تريد تنفيذ هذه التهيئة في بيئة سطر الأوامر، أعد مباشرةً هنا
            return;
        }
        
        // تنفيذ كل 10 ثوانٍ
        \Workerman\Timer::add(10, function () {
            // من أجل التوضيح، نستخدم المخرجات بدل عملية الإبلاغ
            echo memory_get_usage() . "\n";
        });
        
    }

}
```

> **تلميح**
> عند استخدام سطر الأوامر، سينفّذ الإطار أيضًا طريقة start المعرفة في `config/bootstrap.php`، ويمكننا التحقق مما إذا كانت البيئة هي سطر الأوامر عبر التحقق من كون `$worker` يساوي null، ومن ثمّ نقرر ما إذا كنا سننفذ كود تهيئة الأعمال أم لا.

#### الإعداد لبدء العملية
افتح `config/bootstrap.php` وأضف فئة `MemReport` إلى عناصر البدء.
```php
return [
    // ...الإعدادات الأخرى محذوفة هنا...
    
    app\bootstrap\MemReport::class,
];
```

بهذا نكون قد أكملنا إجراء تهيئة الأعمال.

## توضيحات إضافية
تقوم [العمليات المخصصة](../process.md) بعد بدئها بتنفيذ طريقة start المعرفة في `config/bootstrap.php`. يمكننا استخدام `$worker->name` لتحديد العملية الحالية، واستخدام `$worker->id` لتحديد رقم العملية، ومن ثمّ نقرر تنفيذ كود تهيئة الأعمال في تلك العملية أم لا. فعلى سبيل المثال، إذا كنا نريد التنفيذ فقط في العملية رقم 0 من webman، فمحتوى `MemReport.php` يكون شبيهًا بالتالي:
```php
<?php

namespace app\bootstrap;

use Webman\Bootstrap;

class MemReport implements Bootstrap
{
    public static function start($worker)
    {
        // هل هي بيئة سطر الأوامر؟
        $is_console = !$worker;
        if ($is_console) {
            // إذا كنت لا تريد تنفيذ هذه التهيئة في بيئة سطر الأوامر، أعد مباشرةً هنا
            return;
        }

        // التنفيذ فقط في العملية رقم 0 من webman
        if ($worker->name != 'webman' || $worker->id != 0) {
            return;
        }
        
        // تنفيذ كل 10 ثوانٍ
        \Workerman\Timer::add(10, function () {
            // من أجل التوضيح، نستخدم المخرجات بدل عملية الإبلاغ
            echo memory_get_usage() . "\n";
        });
        
    }

}
```
