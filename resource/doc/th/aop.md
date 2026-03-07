# AOP

> ขอขอบคุณผู้เขียน Hyperf สำหรับการมีส่วนร่วม

## การติดตั้ง

- ติดตั้ง aop-integration

```shell
composer require "hyperf/aop-integration: ^1.1"
```

## เพิ่มการตั้งค่าที่เกี่ยวข้องกับ AOP

ต้องเพิ่มไฟล์การตั้งค่า `config.php` ในโฟลเดอร์ `config`

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
        // เพิ่ม Aspect ที่เกี่ยวข้องที่นี่
        app\aspect\DebugAspect::class,
    ]
];

```

## กำหนดค่าไฟล์เอนทรี start.php

> วางโค้ดเริ่มต้นไว้ด้านล่างการตั้งค่า timezone โค้ดส่วนอื่นถูกละไว้ด้านล่าง

```
use Hyperf\AopIntegration\ClassLoader;

if ($timezone = config('app.default_timezone')) {
    date_default_timezone_set($timezone);
}

// การเริ่มต้น
ClassLoader::init();
```

## การทดสอบ

ขั้นแรก สร้างคลาสที่จะถูกดักจับ:

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

จากนั้นเพิ่ม `DebugAspect` ที่เกี่ยวข้อง:

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

จากนั้นแก้ไขตัวควบคุม `app/controller/IndexController.php`:

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

จากนั้นกำหนดค่าเส้นทาง:

```php
<?php
use Webman\Route;

Route::any('/json', [app\controller\IndexController::class, 'json']);
```

สุดท้าย เริ่มต้นบริการและรันการทดสอบ:

```shell
php start.php start
curl  http://127.0.0.1:8787/json
```
