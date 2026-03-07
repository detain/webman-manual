# Redis

[webman/redis](https://github.com/webman-php/redis) estende [illuminate/redis](https://github.com/illuminate/redis) con un pool di connessioni e supporta ambienti sia con che senza coroutine. L’uso è identico a Laravel.

Prima di usare `illuminate/redis`, è necessario installare l’estensione redis per `php-cli`.

## Installazione

```php
composer require -W webman/redis illuminate/events
```

Dopo l’installazione è necessario riavviare (il reload non è sufficiente).

## Configurazione

Il file di configurazione Redis è in `config/redis.php`:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // Pool di connessioni
            'max_connections' => 10,     // Numero massimo di connessioni nel pool
            'min_connections' => 1,      // Numero minimo di connessioni nel pool
            'wait_timeout' => 3,         // Tempo massimo di attesa per ottenere una connessione (secondi)
            'idle_timeout' => 50,        // Connessioni rilasciate dopo questo tempo fino a min_connections
            'heartbeat_interval' => 50,  // Intervallo heartbeat (non superare 60 secondi)
        ],
    ]
];
```

## Pool di connessioni

* Ogni processo ha il proprio pool; i pool non sono condivisi tra processi.
* Senza coroutine l’esecuzione è sequenziale, quindi si usa al massimo una connessione.
* Con coroutine l’esecuzione è concorrente e il pool si adatta tra `min_connections` e `max_connections`.
* Se le coroutine che usano Redis superano `max_connections`, aspettano fino a `wait_timeout` secondi; oltre si genera un’eccezione.
* In stato idle (con o senza coroutine), le connessioni vengono rilasciate dopo `idle_timeout` fino a `min_connections` (0 consentito).

## Esempio

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

## Interfaccia Redis

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
> Usare l’interfaccia `Redis::select($db)` con cautela. Essendo webman un framework residente in memoria, il cambio di database in una richiesta influisce sulle successive. Per più database, si consiglia di configurare connessioni Redis distinte per ogni `$db`.

## Utilizzare più connessioni Redis

Esempio nel file `config/redis.php`:

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

Di default si usa la connessione in `default`. Usare `Redis::connection()` per scegliere quale connessione Redis usare:

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## Configurazione del cluster

Se l’applicazione usa un cluster Redis, definirlo nella configurazione con la chiave `clusters`:

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

Per default il cluster applica lo sharding lato client sui nodi, consentendo pool e molta memoria. Lo sharding lato client non gestisce i guasti; è adatto soprattutto per dati in cache provenienti da un’altra base principale. Per il cluster nativo Redis, specificare in `options` nella configurazione:

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

## Comandi pipeline

Per inviare molti comandi in una sola operazione usare il pipeline. Il metodo `pipeline` accetta una closure; tutti i comandi vengono eseguiti in un’unica operazione:

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
