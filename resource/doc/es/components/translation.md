# Multilingüe

La funcionalidad multilingüe utiliza el componente [symfony/translation](https://github.com/symfony/translation).

## Instalación
```
composer require symfony/translation
```

## Crear paquetes de idioma
webman guarda los paquetes de idioma por defecto en el directorio `resource/translations` (crear si no existe). Para cambiar el directorio, configúralo en `config/translation.php`.
Cada idioma corresponde a una subcarpeta y las definiciones se guardan en `messages.php` por defecto. Ejemplo:
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

Todos los archivos de idioma devuelven un array, por ejemplo:
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## Configuración

`config/translation.php`

```php
return [
    // Idioma por defecto
    'locale' => 'zh_CN',
    // Idioma alternativo: si no se encuentra la traducción en el idioma actual, se intenta con el alternativo
    'fallback_locale' => ['zh_CN', 'en'],
    // Carpeta donde se guardan los archivos de idioma
    'path' => base_path() . '/resource/translations',
];
```

## Traducción

Se usa el método `trans()` para traducir.

Crear archivo de idioma `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 世界!',
];
```

Crear archivo `app/controller/UserController.php`:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        $hello = trans('hello'); // 你好 世界!
        return response($hello);
    }
}
```

Al acceder a `http://127.0.0.1:8787/user/get` se devolverá "你好 世界!"

## Cambiar el idioma por defecto

Usa el método `locale()` para cambiar de idioma.

Añadir archivo de idioma `resource/translations/en/messages.php`:
```php
return [
    'hello' => 'hello world!',
];
```

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // Cambiar idioma
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
Al acceder a `http://127.0.0.1:8787/user/get` se devolverá "hello world!"

También puedes usar el cuarto parámetro de `trans()` para cambiar el idioma temporalmente. El ejemplo anterior equivale a:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // El 4º parámetro cambia el idioma
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## Establecer el idioma explícitamente en cada petición
translation es un singleton, por lo que todas las peticiones comparten la misma instancia. Si una petición usa `locale()` para establecer el idioma por defecto, afectará a todas las peticiones posteriores del proceso. Por eso conviene establecer el idioma explícitamente en cada petición, por ejemplo con el siguiente middleware:

Crear archivo `app/middleware/Lang.php` (crear el directorio si no existe):
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Lang implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        locale(session('lang', 'zh_CN'));
        return $handler($request);
    }
}
```

Añadir el middleware global en `config/middleware.php`:
```php
return [
    // Middleware global
    '' => [
        // ... otros middlewares omitidos
        app\middleware\Lang::class,
    ]
];
```


## Usar marcadores de posición
A veces un mensaje contiene variables que deben traducirse, por ejemplo
```php
trans('hello ' . $name);
```
En estos casos se usan marcadores de posición.

Actualizar `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 %name%!',
];
```
Al traducir, se pasan los valores mediante el segundo parámetro:
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## Manejar plurales
En algunos idiomas la estructura de la oración varía según la cantidad. Por ejemplo, "There is %count% apple" es correcto cuando `%count%` es 1, pero incorrecto cuando es mayor.

En estos casos se usa el **pipe** (`|`) para listar las formas en plural.

Añadir `apple_count` en `resource/translations/en/messages.php`:
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

También se pueden especificar rangos numéricos para reglas de plural más complejas:
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## Especificar archivo de idioma

Por defecto el archivo se llama `messages.php`, pero puedes crear archivos con otros nombres.

Crear archivo `resource/translations/zh_CN/admin.php`:
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

Especificar el archivo con el tercer parámetro de `trans()` (sin la extensión `.php`):
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## Más información
Consultar [documentación de symfony/translation](https://symfony.com/doc/current/translation.html)
