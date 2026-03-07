# Redis
Redisの使い方はデータベースと同様です。例：`plugin/foo/config/redis.php`
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

使用時
```php
use support\Redis;
Redis::connection('plugin.foo.default')->get('key');
Redis::connection('plugin.foo.cache')->get('key');
```

同様に、メインプロジェクトのRedis設定を再利用する場合
```php
use support\Redis;
Redis::get('key');
// メインプロジェクトにcache接続も設定されていると仮定
Redis::connection('cache')->get('key');
```
