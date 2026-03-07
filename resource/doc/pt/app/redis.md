# Redis
O uso do Redis é semelhante ao do banco de dados, por exemplo `plugin/foo/config/redis.php`
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

Ao usar
```php
use support\Redis;
Redis::connection('plugin.foo.default')->get('key');
Redis::connection('plugin.foo.cache')->get('key');
```

Do mesmo modo, se quiser reutilizar a configuração Redis do projeto principal
```php
use support\Redis;
Redis::get('key');
// Supondo que o projeto principal também configurou uma conexão de cache
Redis::connection('cache')->get('key');
```
