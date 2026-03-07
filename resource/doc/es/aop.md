# AOP

> Gracias al autor de Hyperf por su contribución.

## Instalación

- Instalar aop-integration

```shell
composer require "hyperf/aop-integration: ^1.1"
```

## Añadir configuración relacionada con AOP

Es necesario añadir el archivo de configuración `config.php` en el directorio `config`.

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
        // Aquí se añaden los Aspect correspondientes
        app\aspect\DebugAspect::class,
    ]
];

```

## Configurar el archivo de entrada start.php

> Coloque el código de inicialización debajo de la configuración de zona horaria. El resto del código se omite a continuación.

```
use Hyperf\AopIntegration\ClassLoader;

if ($timezone = config('app.default_timezone')) {
    date_default_timezone_set($timezone);
}

// Inicialización
ClassLoader::init();
```

## Pruebas

Primero, creemos la clase a interceptar:

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

A continuación, añada el `DebugAspect` correspondiente:

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

Luego, edite el controlador `app/controller/IndexController.php`:

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

Después configure la ruta:

```php
<?php
use Webman\Route;

Route::any('/json', [app\controller\IndexController::class, 'json']);
```

Por último, inicie el servicio y ejecute la prueba:

```shell
php start.php start
curl  http://127.0.0.1:8787/json
```
