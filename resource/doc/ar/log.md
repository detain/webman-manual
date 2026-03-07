# السجلات (Log)
يستخدم webman مكتبة [monolog/monolog](https://github.com/Seldaek/monolog) لمعالجة السجلات.

## الاستخدام
```php
<?php
namespace app\controller;

use support\Request;
use support\Log;

class FooController
{
    public function index(Request $request)
    {
        Log::info('log test');
        return response('hello index');
    }
}
```

## الطرق المتاحة
```php
Log::log($level, $message, array $context = [])
Log::debug($message, array $context = [])
Log::info($message, array $context = [])
Log::notice($message, array $context = [])
Log::warning($message, array $context = [])
Log::error($message, array $context = [])
Log::critical($message, array $context = [])
Log::alert($message, array $context = [])
Log::emergency($message, array $context = [])
```
يعادل
```php
$log = Log::channel('default');
$log->log($level, $message, array $context = [])
$log->debug($message, array $context = [])
$log->info($message, array $context = [])
$log->notice($message, array $context = [])
$log->warning($message, array $context = [])
$log->error($message, array $context = [])
$log->critical($message, array $context = [])
$log->alert($message, array $context = [])
$log->emergency($message, array $context = [])
```

## الإعدادات
```php
return [
    // قناة السجل الافتراضية
    'default' => [
        // معالجات القناة الافتراضية، يمكن تعيين عدة معالجات
        'handlers' => [
            [   
                // اسم فئة المعالج
                'class' => Monolog\Handler\RotatingFileHandler::class,
                // معاملات مُنشئ فئة المعالج
                'constructor' => [
                    runtime_path() . '/logs/webman.log',
                    Monolog\Logger::DEBUG,
                ],
                // متعلق بالتنسيق
                'formatter' => [
                    // اسم فئة معالج التنسيق
                    'class' => Monolog\Formatter\LineFormatter::class,
                    // معاملات مُنشئ فئة معالج التنسيق
                    'constructor' => [ null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
];
```

## قنوات متعددة
يدعم monolog القنوات المتعددة، ويستخدم قناة `default` افتراضياً. إذا أردت إضافة قناة `log2`، يكون الإعداد مشابهاً لما يلي:
```php
return [
    // قناة السجل الافتراضية
    'default' => [
        // معالجات القناة الافتراضية، يمكن تعيين عدة معالجات
        'handlers' => [
            [   
                // اسم فئة المعالج
                'class' => Monolog\Handler\RotatingFileHandler::class,
                // معاملات مُنشئ فئة المعالج
                'constructor' => [
                    runtime_path() . '/logs/webman.log',
                    Monolog\Logger::DEBUG,
                ],
                // متعلق بالتنسيق
                'formatter' => [
                    // اسم فئة معالج التنسيق
                    'class' => Monolog\Formatter\LineFormatter::class,
                    // معاملات مُنشئ فئة معالج التنسيق
                    'constructor' => [ null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
    // قناة log2
    'log2' => [
        // معالجات قناة log2، يمكن تعيين عدة معالجات
        'handlers' => [
            [   
                // اسم فئة المعالج
                'class' => Monolog\Handler\RotatingFileHandler::class,
                // معاملات مُنشئ فئة المعالج
                'constructor' => [
                    runtime_path() . '/logs/log2.log',
                    Monolog\Logger::DEBUG,
                ],
                // متعلق بالتنسيق
                'formatter' => [
                    // اسم فئة معالج التنسيق
                    'class' => Monolog\Formatter\LineFormatter::class,
                    // معاملات مُنشئ فئة معالج التنسيق
                    'constructor' => [ null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
];
```

استخدام قناة `log2` كالتالي:
```php
<?php
namespace app\controller;

use support\Request;
use support\Log;

class FooController
{
    public function index(Request $request)
    {
        $log = Log::channel('log2');
        $log->info('log2 test');
        return response('hello index');
    }
}
```
