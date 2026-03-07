# Redis

[webman/redis](https://github.com/webman-php/redis) estende [illuminate/redis](https://github.com/illuminate/redis) adicionando pool de conexões e suporta ambientes com e sem coroutines. O uso é o mesmo do Laravel.

Antes de usar `illuminate/redis`, é necessário instalar a extensão redis para `php-cli`.

## Instalação

```php
composer require -W webman/redis illuminate/events
```

Após a instalação, é necessário reiniciar (reload não funciona).

## Configuração

O arquivo de configuração do Redis está em `config/redis.php`:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // Pool de conexões
            'max_connections' => 10,     // Máximo de conexões no pool
            'min_connections' => 1,      // Mínimo de conexões no pool
            'wait_timeout' => 3,         // Tempo máximo de espera ao obter conexão (segundos)
            'idle_timeout' => 50,        // Após esse tempo, conexões são liberadas até min_connections
            'heartbeat_interval' => 50,  // Intervalo de heartbeat (não exceder 60 segundos)
        ],
    ]
];
```

## Pool de conexões

* Cada processo tem seu próprio pool; os pools não são compartilhados entre processos.
* Sem coroutines a execução é sequencial, então no máximo uma conexão é usada.
* Com coroutines a execução é concorrente e o pool escala entre `min_connections` e `max_connections`.
* Se o número de coroutines usando Redis exceder `max_connections`, elas aguardam até `wait_timeout` segundos; além disso uma exceção é lançada.
* Em idle (com ou sem coroutines), as conexões são liberadas após `idle_timeout` até alcançar `min_connections` (0 permitido).

## Exemplo

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

## Interface Redis

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

> **Atenção**
> Use a interface `Redis::select($db)` com cuidado. Como o webman é um framework residente em memória, mudar de banco em uma requisição afeta as seguintes. Para múltiplos bancos, configure conexões Redis separadas para cada `$db`.

## Usar várias conexões Redis

Exemplo no arquivo `config/redis.php`:

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

Por padrão usa-se a conexão em `default`. Use `Redis::connection()` para escolher qual conexão Redis usar:

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## Configuração de cluster

Se a aplicação usa um cluster Redis, defina-o na configuração com a chave `clusters`:

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

Por padrão o cluster faz sharding no cliente nos nós, permitindo pools e grande quantidade de memória. O sharding no cliente não trata falhas; é indicado principalmente para dados em cache de outro banco principal. Para o cluster nativo do Redis, especifique em `options` na configuração:

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

Quando precisar enviar muitos comandos em uma única operação, use o pipeline. O método `pipeline` aceita um closure; todos os comandos são executados em uma única operação:

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
