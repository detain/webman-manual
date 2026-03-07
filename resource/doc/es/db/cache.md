# Caché

[webman/cache](https://github.com/webman-php/cache) es un componente de caché basado en [symfony/cache](https://github.com/symfony/cache), compatible con entornos con y sin corutinas, y con soporte de grupo de conexiones.

## Instalación

```php
composer require -W webman/cache
```

## Ejemplo
```php
<?php
namespace app\controller;

use support\Request;
use support\Cache;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Cache::set($key, rand());
        return response(Cache::get($key));
    }
}
```

## Ubicación del archivo de configuración
El archivo de configuración está en `config/cache.php`. Créelo manualmente si no existe.

## Contenido del archivo de configuración
```php
<?php
return [
    'default' => 'file',
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default'
        ],
        'array' => [
            'driver' => 'array'
        ],
        'apcu' => [
            'driver' => 'apcu'
        ]
    ]
];
```
`stores.driver` admite cuatro controladores: **file**, **redis**, **array** y **apcu**.

#### controlador file
Es el controlador por defecto. No tiene dependencias externas. Permite compartir caché entre procesos. No permite compartir entre varios servidores.

#### controlador array
Almacenamiento en memoria con el mejor rendimiento, pero consume memoria. No permite compartir entre procesos ni servidores. Los datos se pierden al reiniciar el proceso. Suele usarse en proyectos con poco volumen de caché.

#### controlador apcu
Almacenamiento en memoria. Su rendimiento solo es superado por array. Permite compartir caché entre procesos. No permite compartir entre varios servidores. Los datos se pierden al reiniciar el proceso. Suele usarse en proyectos con poco volumen de caché.

> Se requiere instalar y habilitar la [extensión APCu](https://pecl.php.net/package/APCu). No se recomienda en escenarios con escrituras/eliminaciones frecuentes de caché, ya que puede provocar una degradación notable del rendimiento.

#### controlador redis
Depende del componente [webman/redis](./redis.md). Permite compartir caché entre procesos y servidores.

**stores.redis.connection**

`stores.redis.connection` corresponde a la clave definida en `config/redis.php`. Al usar Redis, reutiliza la configuración de `webman/redis`, incluido el grupo de conexiones.

**Se recomienda añadir una configuración de Redis dedicada en `config/redis.php` para caché, por ejemplo:**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== Añadir nuevo
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

Luego asigne `stores.redis.connection` a `cache`. El `config/cache.php` final debería verse así:

```php
<?php
return [
    'default' => 'redis', // <====
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache' // <====
        ],
        'array' => [
            'driver' => 'array'
        ]
    ]
];
```

## Cambio de almacenamiento
Puede cambiar manualmente el almacenamiento para usar distintos controladores, por ejemplo:

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **Consejo**
> Los nombres de las claves de caché están restringidos por [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) y no deben contener ninguno de estos caracteres: `{}()/\@:`. A partir de `symfony/cache` 7.2.4, esta comprobación puede evitarse configurando en PHP ini la opción `zend.assertions=-1`.

## Uso de otros componentes de caché

Consulte [Other Databases](others.md#ThinkCache) para el componente [ThinkCache](https://github.com/webman-php/think-cache).
