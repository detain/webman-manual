# Redis

[webman/redis](https://github.com/webman-php/redis) erweitert [illuminate/redis](https://github.com/illuminate/redis) um einen Verbindungspool und unterstützt sowohl Coroutine- als auch Nicht-Coroutine-Umgebungen. Die Verwendung entspricht Laravel.

Vor der Verwendung von `illuminate/redis` muss die Redis-Erweiterung für `php-cli` installiert sein.

## Installation

```php
composer require -W webman/redis illuminate/events
```

Nach der Installation ist ein Neustart erforderlich (Reload reicht nicht).

## Konfiguration

Die Redis-Konfiguration liegt unter `config/redis.php`:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // Verbindungspool
            'max_connections' => 10,     // Maximale Verbindungen im Pool
            'min_connections' => 1,      // Minimale Verbindungen im Pool
            'wait_timeout' => 3,          // Maximale Wartezeit beim Erlangen einer Verbindung (Sekunden)
            'idle_timeout' => 50,         // Leerlauf-Timeout; Verbindungen werden geschlossen bis min_connections erreicht ist
            'heartbeat_interval' => 50,  // Heartbeat-Intervall (nicht über 60 Sekunden)
        ],
    ]
];
```

## Verbindungspool

* Jeder Prozess hat seinen eigenen Pool; Pools werden nicht zwischen Prozessen geteilt.
* Ohne Coroutine läuft alles sequenziell, sodass maximal eine Verbindung genutzt wird.
* Mit Coroutine laufen Anforderungen parallel und der Pool skaliert dynamisch zwischen `min_connections` und `max_connections`.
* Überschreitet die Zahl der Redis nutzenden Coroutines `max_connections`, warten sie bis zu `wait_timeout` Sekunden; danach wird eine Exception ausgelöst.
* Im Leerlauf (mit oder ohne Coroutine) werden Verbindungen nach `idle_timeout` freigegeben, bis `min_connections` erreicht ist (0 ist erlaubt).

## Beispiel

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

## Redis-API

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

Entspricht:

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

> **Hinweis**
> Die Nutzung von `Redis::select($db)` sollte vermieden werden. Da webman im Speicher persistent läuft, beeinflusst ein Wechsel der Datenbank die folgenden Anfragen. Für mehrere Datenbanken sind separate Redis-Verbindungen pro `$db` zu empfehlen.

## Mehrere Redis-Verbindungen

Beispiel in `config/redis.php`:

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

Standardmäßig wird die Verbindung unter `default` genutzt. Mit `Redis::connection()` wählen Sie die gewünschte Redis-Verbindung:

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## Cluster-Konfiguration

Bei Verwendung von Redis-Clustern diese in der Konfiguration unter dem Schlüssel `clusters` definieren:

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

Standardmäßig nutzt der Cluster Client-seitiges Sharding auf den Knoten und ermöglicht Pools mit großer Speicherkapazität. Client-Sharding behandelt keine Fehlerfälle; es eignet sich vor allem für Cache-Daten aus einer anderen primären Datenbank. Für den nativen Redis-Cluster in der Konfiguration unter `options` angeben:

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

## Pipeline-Befehle

Für viele Befehle in einem Durchlauf eignet sich die Pipeline. Die Methode `pipeline` akzeptiert eine Closure; alle Befehle werden in einem Vorgang ausgeführt:

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
