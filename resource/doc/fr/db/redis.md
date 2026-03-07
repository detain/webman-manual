# Redis

[webman/redis](https://github.com/webman-php/redis) étend [illuminate/redis](https://github.com/illuminate/redis) avec un pool de connexions et fonctionne en environnement avec ou sans coroutines. L’usage est identique à Laravel.

L’extension redis doit être installée pour `php-cli` avant d’utiliser `illuminate/redis`.

## Installation

```php
composer require -W webman/redis illuminate/events
```

Un redémarrage est nécessaire après l’installation (un simple reload ne suffit pas).

## Configuration

Le fichier de configuration Redis se trouve dans `config/redis.php` :

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // Pool de connexions
            'max_connections' => 10,     // Nombre maximal de connexions
            'min_connections' => 1,      // Nombre minimal de connexions
            'wait_timeout' => 3,         // Délai max pour obtenir une connexion (secondes)
            'idle_timeout' => 50,        // Au-delà, les connexions sont libérées jusqu’à min_connections
            'heartbeat_interval' => 50,  // Intervalle des heartbeats (ne pas dépasser 60 secondes)
        ],
    ]
];
```

## Pool de connexions

* Chaque processus dispose de son propre pool ; ils ne sont pas partagés entre processus.
* Sans coroutines, l’exécution est séquentielle, donc au plus une connexion est utilisée.
* Avec coroutines, l’exécution est parallèle et le pool s’adapte entre `min_connections` et `max_connections`.
* Si le nombre de coroutines utilisant Redis dépasse `max_connections`, elles attendent au maximum `wait_timeout` secondes, puis une exception est levée.
* En inactivité (avec ou sans coroutines), les connexions sont libérées après `idle_timeout` jusqu’à atteindre `min_connections` (0 autorisé).

## Exemple

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

## API Redis

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

Équivalent à :

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

> **Remarque**
> Utilisez `Redis::select($db)` avec précaution : webman restant en mémoire, un changement de base dans une requête affecte les suivantes. Pour plusieurs bases, configurez une connexion Redis distincte par `$db`.

## Utiliser plusieurs connexions Redis

Exemple dans `config/redis.php` :

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

Par défaut, la connexion définie sous `default` est utilisée. Utilisez `Redis::connection()` pour choisir la connexion Redis :

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## Configuration du cluster

Si l’application utilise un cluster Redis, définissez-le dans la configuration sous la clé `clusters` :

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

Par défaut, le cluster effectue un sharding côté client sur les nœuds, permettant des pools et beaucoup de mémoire disponible. Le sharding côté client ne gère pas les pannes ; il convient surtout pour des données en cache provenant d’une autre base principale. Pour un cluster Redis natif, précisez dans `options` de la configuration :

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

## Commandes en pipeline

Pour envoyer beaucoup de commandes en une seule opération, utilisez le pipeline. La méthode `pipeline` accepte une closure ; toutes les commandes sont exécutées en un bloc :

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
