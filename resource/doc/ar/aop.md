# AOP

> شكراً لمؤلف Hyperf على المساهمة.

## التثبيت

- تثبيت aop-integration

```shell
composer require "hyperf/aop-integration: ^1.1"
```

## إضافة إعدادات AOP

يجب إضافة ملف الإعدادات `config.php` في مجلد `config`.

```php
<?php

use Hyperf\Di\Annotation\AspectCollector;

return [
    'annotations' => [
        'scan' => [
            'paths' => [
                BASE_PATH . '/app',
            ],
            'ignore_annotations' => [
                'mixin',
            ],
            'class_map' => [
            ],
            'collectors' => [
                AspectCollector::class
            ],
        ],
    ],
    'aspects' => [
        // أضف الـ Aspect المقابل هنا
        app\aspect\DebugAspect::class,
    ]
];

```

## تكوين ملف الدخول start.php

> ضع كود التهيئة أسفل إعداد timezone. باقي الكود مُحذوف أدناه.

```
use Hyperf\AopIntegration\ClassLoader;

if ($timezone = config('app.default_timezone')) {
    date_default_timezone_set($timezone);
}

// التهيئة
ClassLoader::init();
```

## الاختبار

أولاً، لننشئ الفئة التي سيتم اعتراضها:

```php
<?php
namespace app\service;

class UserService
{
    public function first(): array
    {
        return ['id' => 1];
    }
}
```

ثم أضف الـ `DebugAspect` المقابل:

```php
<?php
namespace app\aspect;

use app\service\UserService;
use Hyperf\Di\Aop\AbstractAspect;
use Hyperf\Di\Aop\ProceedingJoinPoint;

class DebugAspect extends AbstractAspect
{
    public $classes = [
        UserService::class . '::first',
    ];

    public function process(ProceedingJoinPoint $proceedingJoinPoint)
    {
        var_dump(11);
        return $proceedingJoinPoint->process();
    }
}
```

بعد ذلك، عدّل المتحكم `app/controller/IndexController.php`:

```php
<?php
namespace app\controller;

use app\service\UserService;
use support\Request;

class IndexController
{
    public function json(Request $request)
    {
        return json(['code' => 0, 'msg' => 'ok', 'data' => (new UserService())->first()]);
    }
}
```

ثم حدّث المسار:

```php
<?php
use Webman\Route;

Route::any('/json', [app\controller\IndexController::class, 'json']);
```

أخيراً، شغّل الخدمة ونفّذ الاختبار:

```shell
php start.php start
curl  http://127.0.0.1:8787/json
```
