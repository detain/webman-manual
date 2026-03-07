# AOP

> Ringraziamenti all'autore di Hyperf per il contributo.

## Installazione

- Installare aop-integration

```shell
composer require "hyperf/aop-integration: ^1.1"
```

## Aggiungere la configurazione AOP

È necessario aggiungere il file di configurazione `config.php` nella directory `config`.

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
        // Inserire qui l'Aspect corrispondente
        app\aspect\DebugAspect::class,
    ]
];

```

## Configurare il file di ingresso start.php

> Posizionare il codice di inizializzazione sotto l'impostazione timezone. Il resto del codice è omesso di seguito.

```
use Hyperf\AopIntegration\ClassLoader;

if ($timezone = config('app.default_timezone')) {
    date_default_timezone_set($timezone);
}

// Inizializzazione
ClassLoader::init();
```

## Test

Prima creiamo la classe da intercettare:

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

Poi aggiungere il `DebugAspect` corrispondente:

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

Successivamente, modificare il controller `app/controller/IndexController.php`:

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

Poi configurare la route:

```php
<?php
use Webman\Route;

Route::any('/json', [app\controller\IndexController::class, 'json']);
```

Infine, avviare il servizio ed eseguire il test:

```shell
php start.php start
curl  http://127.0.0.1:8787/json
```
