# AOP

> Dank an den Autor von Hyperf für den Beitrag.

## Installation

- aop-integration installieren

```shell
composer require "hyperf/aop-integration: ^1.1"
```

## AOP-Konfiguration hinzufügen

Die `config.php` Konfigurationsdatei muss im `config` Verzeichnis angelegt werden.

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
        // Hier den entsprechenden Aspect eintragen
        app\aspect\DebugAspect::class,
    ]
];

```

## Einstiegsdatei start.php konfigurieren

> Die Initialisierung wird unter der timezone-Einstellung platziert. Anderer Code wird im Folgenden weggelassen.

```
use Hyperf\AopIntegration\ClassLoader;

if ($timezone = config('app.default_timezone')) {
    date_default_timezone_set($timezone);
}

// Initialisierung
ClassLoader::init();
```

## Testen

Zuerst erstellen wir die zu interceptende Klasse:

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

Dann fügen wir den entsprechenden `DebugAspect` hinzu:

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

Anschließend den Controller `app/controller/IndexController.php` bearbeiten:

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

Dann die Route konfigurieren:

```php
<?php
use Webman\Route;

Route::any('/json', [app\controller\IndexController::class, 'json']);
```

Zum Schluss den Dienst starten und testen:

```shell
php start.php start
curl  http://127.0.0.1:8787/json
```
