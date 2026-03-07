# Redis

[webman/redis](https://github.com/webman-php/redis) extiende [illuminate/redis](https://github.com/illuminate/redis) añadiendo un pool de conexiones y funciona tanto en entornos con corutinas como sin ellas. Su uso es el mismo que en Laravel.

Debe instalar la extensión redis para `php-cli` antes de usar `illuminate/redis`.

## Instalación

```php
composer require -W webman/redis illuminate/events
```

Tras la instalación, es necesario reiniciar (un reload no es suficiente).

## Configuración

El archivo de configuración de Redis está en `config/redis.php`:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // Pool de conexiones
            'max_connections' => 10,     // Máximo de conexiones en el pool
            'min_connections' => 1,     // Mínimo de conexiones en el pool
            'wait_timeout' => 3,         // Tiempo máximo de espera al obtener una conexión (segundos)
            'idle_timeout' => 50,        // Tiempo de inactividad; tras él se liberan hasta llegar a min_connections
            'heartbeat_interval' => 50, // Intervalo de heartbeat (no superar 60 segundos)
        ],
    ]
];
```

## Pool de conexiones

* Cada proceso tiene su propio pool; no se comparten entre procesos.
* Sin corutinas la ejecución es secuencial, por lo que se usa como máximo una conexión.
* Con corutinas la ejecución es concurrente y el pool se ajusta entre `min_connections` y `max_connections`.
* Si el número de corutinas que usan Redis supera `max_connections`, esperan hasta `wait_timeout` segundos; después se lanza una excepción.
* En inactividad (con o sin corutinas), las conexiones se liberan tras `idle_timeout` hasta alcanzar `min_connections` (0 permitido).

## Ejemplo

```php
<?php
namespace app\controller;

use support\Request;
use support\Redis;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Redis::set($key, rand());
        return response(Redis::get($key));
    }
}
```

## API de Redis

```php
Redis::append($key, $value)
Redis::bitCount($key)
Redis::decr($key, $value)
Redis::decrBy($key, $value)
Redis::get($key)
Redis::getBit($key, $offset)
Redis::getRange($key, $start, $end)
Redis::getSet($key, $value)
Redis::incr($key, $value)
Redis::incrBy($key, $value)
Redis::incrByFloat($key, $value)
Redis::mGet(array $keys)
Redis::getMultiple(array $keys)
Redis::mSet($pairs)
Redis::mSetNx($pairs)
Redis::set($key, $value, $expireResolution = null, $expireTTL = null, $flag = null)
Redis::setBit($key, $offset, $value)
Redis::setEx($key, $ttl, $value)
Redis::pSetEx($key, $ttl, $value)
Redis::setNx($key, $value)
Redis::setRange($key, $offset, $value)
Redis::strLen($key)
Redis::del(...$keys)
Redis::exists(...$keys)
Redis::expire($key, $ttl)
Redis::expireAt($key, $timestamp)
Redis::select($dbIndex)
```

Equivalente a:

```php
$redis = Redis::connection('default');
$redis->append($key, $value)
$redis->bitCount($key)
$redis->decr($key, $value)
$redis->decrBy($key, $value)
$redis->get($key)
$redis->getBit($key, $offset)
...
```

> **Nota**
> Use `Redis::select($db)` con cuidado. Al ser webman un framework residente en memoria, cambiar de base en una petición afecta a las siguientes. Para varias bases, configure conexiones Redis separadas por cada `$db`.

## Usar varias conexiones Redis

Ejemplo en `config/redis.php`:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
    ],

    'cache' => [
        'host'     => '127.0.0.1',
        'password' => null,
        'port'     => 6379,
        'database' => 1,
    ],

]
```

Por defecto se usa la conexión de `default`. Use `Redis::connection()` para elegir la conexión Redis:

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## Configuración de clúster

Si la aplicación usa un clúster Redis, defínalo en la configuración con la clave `clusters`:

```php
return [
    'clusters' => [
        'default' => [
            [
                'host'     => 'localhost',
                'username' => null,
                'password' => null,
                'port'     => 6379,
                'database' => 0,
            ],
        ],
    ],

];
```

Por defecto el clúster hace sharding en el cliente en los nodos, permitiendo pools y gran cantidad de memoria. El sharding en cliente no gestiona fallos; conviene para datos de caché de otra base principal. Para clúster nativo de Redis, especifique en `options` de la configuración:

```php
return[
    'options' => [
        'cluster' => 'redis',
    ],

    'clusters' => [
        // ...
    ],
];
```

## Comandos pipeline

Para enviar muchos comandos en una sola operación, use el pipeline. El método `pipeline` acepta un closure; todos los comandos se ejecutan en un solo bloque:

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
