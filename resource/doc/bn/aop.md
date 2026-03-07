# AOP

> Hyperf লেখকের অবদানের জন্য ধন্যবাদ।

## ইনস্টলেশন

- aop-integration ইনস্টল করুন

```shell
composer require "hyperf/aop-integration: ^1.1"
```

## AOP-সম্পর্কিত কনফিগারেশন যোগ করুন

config ফোল্ডারে config.php কনফিগারেশন ফাইল যোগ করতে হবে।

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
        // এখানে সংশ্লিষ্ট Aspect যোগ করুন
        app\aspect\DebugAspect::class,
    ]
];

```

## এন্ট্রি ফাইল start.php কনফিগার করুন

> timezone সেটিং এর নিচে ইনিশিয়ালাইজেশন কোড রাখুন। নিচে অন্যান্য কোড বাদ দেওয়া হয়েছে।

```
use Hyperf\AopIntegration\ClassLoader;

if ($timezone = config('app.default_timezone')) {
    date_default_timezone_set($timezone);
}

// ইনিশিয়ালাইজেশন
ClassLoader::init();
```

## পরীক্ষা

প্রথমে ইন্টারসেপ্ট করার ক্লাস তৈরি করুন:

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

তারপর সংশ্লিষ্ট DebugAspect যোগ করুন:

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

এরপর কন্ট্রোলার app/controller/IndexController.php সম্পাদনা করুন:

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

তারপর রাউট কনফিগার করুন:

```php
<?php
use Webman\Route;

Route::any('/json', [app\controller\IndexController::class, 'json']);
```

শেষে সার্ভিস চালু করুন এবং পরীক্ষা চালান:

```shell
php start.php start
curl  http://127.0.0.1:8787/json
```
