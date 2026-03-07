# Redis
Использование Redis аналогично базе данных, например `plugin/foo/config/redis.php`
```php
return [
    'default' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 1,
    ],
];
```
При использовании
```php
use support\Redis;
Redis::connection('plugin.foo.default')->get('key');
Redis::connection('plugin.foo.cache')->get('key');
```

Аналогично, если нужно переиспользовать конфигурацию Redis основного проекта
```php
use support\Redis;
Redis::get('key');
// Предполагая, что в основном проекте также настроено подключение cache
Redis::connection('cache')->get('key');
```
