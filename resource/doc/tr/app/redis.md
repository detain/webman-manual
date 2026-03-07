# Redis
Redis kullanımı veritabanına benzer, örneğin `plugin/foo/config/redis.php`
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
Kullanırken
```php
use support\Redis;
Redis::connection('plugin.foo.default')->get('key');
Redis::connection('plugin.foo.cache')->get('key');
```

Aynı şekilde, ana projenin Redis yapılandırmasını yeniden kullanmak isterseniz
```php
use support\Redis;
Redis::get('key');
// Ana projede cache bağlantısı da yapılandırıldığını varsayalım
Redis::connection('cache')->get('key');
```
