# AOP

> Hyperf yazarının katkısı için teşekkürler.

## Kurulum

- aop-integration kurulumu

```shell
composer require "hyperf/aop-integration: ^1.1"
```

## AOP ile ilgili yapılandırma ekleme

`config` dizinine `config.php` yapılandırma dosyasını eklememiz gerekiyor.

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
        // Buraya ilgili Aspect'i ekleyin
        app\aspect\DebugAspect::class,
    ]
];

```

## Giriş dosyası start.php yapılandırması

> Başlatma kodunu timezone ayarının altına yerleştirin. Diğer kodlar aşağıda atlanmıştır.

```
use Hyperf\AopIntegration\ClassLoader;

if ($timezone = config('app.default_timezone')) {
    date_default_timezone_set($timezone);
}

// Başlatma
ClassLoader::init();
```

## Test

Önce kesilecek sınıfı oluşturalım:

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

Ardından ilgili `DebugAspect`'i ekleyin:

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

Sonra `app/controller/IndexController.php` denetleyicisini düzenleyin:

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

Ardından rota yapılandırması yapın:

```php
<?php
use Webman\Route;

Route::any('/json', [app\controller\IndexController::class, 'json']);
```

Son olarak servisi başlatın ve testi çalıştırın:

```shell
php start.php start
curl  http://127.0.0.1:8787/json
```
