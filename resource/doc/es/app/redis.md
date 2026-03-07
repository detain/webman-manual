# Redis
El uso de Redis es similar al de una base de datos, por ejemplo `plugin/foo/config/redis.php`
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
Al usar
```php
use support\Redis;
Redis::connection('plugin.foo.default')->get('key');
Redis::connection('plugin.foo.cache')->get('key');
```

Del mismo modo, si se quiere reutilizar la configuración Redis del proyecto principal
```php
use support\Redis;
Redis::get('key');
// Supongamos que el proyecto principal también ha configurado una conexión de cache
Redis::connection('cache')->get('key');
```
