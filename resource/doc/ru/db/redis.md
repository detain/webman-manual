# Redis

[webman/redis](https://github.com/webman-php/redis) расширяет [illuminate/redis](https://github.com/illuminate/redis), добавляя пул соединений и поддерживая окружения как с корутинами, так и без них. Использование такое же, как в Laravel.

Перед использованием `illuminate/redis` необходимо установить расширение redis для `php-cli`.

## Установка

```php
composer require -W webman/redis illuminate/events
```

После установки требуется перезапуск (reload не действует).

## Конфигурация

Файл конфигурации Redis находится в `config/redis.php`:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // Настройки пула соединений
            'max_connections' => 10,     // Максимальное число соединений в пуле
            'min_connections' => 1,      // Минимальное число соединений в пуле
            'wait_timeout' => 3,         // Максимальное время ожидания соединения (секунды)
            'idle_timeout' => 50,        // По истечении простоя соединения освобождаются до min_connections
            'heartbeat_interval' => 50,  // Интервал heartbeat (не более 60 секунд)
        ],
    ]
];
```

## Пул соединений

* У каждого процесса свой пул; пулы не разделяются между процессами.
* Без корутин выполнение последовательное, поэтому используется не более одного соединения.
* С корутинами выполнение параллельное, и пул масштабируется между `min_connections` и `max_connections`.
* Если корутин, использующих Redis, больше, чем `max_connections`, они ждут до `wait_timeout` секунд; затем выбрасывается исключение.
* В простое (с корутинами или без) соединения освобождаются после `idle_timeout`, пока не останется `min_connections` (0 допустимо).

## Пример

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

## Интерфейс Redis

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

Эквивалентно:

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

> **Обратите внимание**
> Будьте осторожны с `Redis::select($db)`. Поскольку webman — постоянный фреймворк в памяти, смена базы в одном запросе влияет на последующие. Для нескольких баз рекомендуется настроить отдельные подключения Redis для каждого `$db`.

## Использование нескольких подключений Redis

Пример в файле `config/redis.php`:

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

По умолчанию используется подключение из `default`. Методом `Redis::connection()` можно выбрать нужное подключение Redis:

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## Конфигурация кластера

Если приложение использует кластер Redis, определите его в конфигурации под ключом `clusters`:

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

По умолчанию кластер может выполнять клиентское шардирование на узлах, что позволяет создавать пулы и использовать много памяти. Клиентское шардирование не обрабатывает сбои; подходит в основном для кеш-данных из другой основной базы. Для нативного кластера Redis укажите в `options` конфигурации:

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

## Команды конвейера

При необходимости отправить много команд за одну операцию используйте pipeline. Метод `pipeline` принимает замыкание; все команды выполняются за один проход:

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
