# Inicio rápido con la base de datos (basado en el componente de Laravel)

[webman/database](https://github.com/webman-php/database) se basa en [illuminate/database](https://github.com/illuminate/database) y añade conexiones en pool para entornos con y sin corutinas. El uso es el mismo que en Laravel.

También puedes consultar [Uso de otros componentes de base de datos](others.md) para utilizar ThinkPHP u otras bases de datos.

## Instalación de la base de datos

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

Tras la instalación es necesario reiniciar la aplicación (reload no sirve).

> **Consejo**
> webman/database depende de `illuminate/database` de Laravel, así que las dependencias se instalan automáticamente.

> **Nota**
> Si no necesitas paginación, eventos de base de datos ni registro de SQL, basta con ejecutar:
> `composer require -W webman/database`

## Configuración de la base de datos
`config/database.php`
```php

return [
    // Base de datos por defecto
    'default' => 'mysql',

    // Configuración de conexiones
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'test',
            'username'    => 'root',
            'password'    => '',
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
            'options' => [
                PDO::ATTR_EMULATE_PREPARES => false, // Obligatorio cuando se usa swoole o swow como runtime
            ],
            'pool' => [ // Configuración del pool de conexiones
                'max_connections' => 5, // Número máximo de conexiones
                'min_connections' => 1, // Número mínimo de conexiones
                'wait_timeout' => 3,    // Tiempo máximo de espera al obtener conexión del pool; excepción si se supera. Solo en entorno con corutinas
                'idle_timeout' => 60,   // Tiempo máximo de inactividad; tras él se cierran hasta min_connections
                'heartbeat_interval' => 50, // Intervalo de latido del pool en segundos; se recomienda menos de 60
            ],
        ],
    ],
];
```

Salvo la configuración de `pool`, el resto coincide con Laravel.

## Sobre el pool de conexiones
* Cada proceso tiene su propio pool; los pools no se comparten entre procesos.
* Sin corutinas las peticiones se ejecutan en serie, no hay concurrencia, así que el pool tendrá como máximo una conexión.
* Con corutinas las peticiones se ejecutan en paralelo; el pool ajusta dinámicamente el número de conexiones, sin superar `max_connections` ni bajar de `min_connections`.
* Como el límite del pool es `max_connections`, cuando hay más corutinas usando la base de datos, algunas esperan en cola hasta `wait_timeout` segundos; si se supera, se lanza una excepción.
* En estado de inactividad (con o sin corutinas), las conexiones se reciclan tras `idle_timeout` hasta llegar a `min_connections` (`min_connections` puede ser 0).


## Ejemplo de uso de la base de datos
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        $default_uid = 29;
        $uid = $request->get('uid', $default_uid);
        $name = Db::table('users')->where('uid', $uid)->value('username');
        return response("hello $name");
    }
}
```

El uso es el mismo que en Laravel: el método `Db::table()` para operar con la base de datos.
