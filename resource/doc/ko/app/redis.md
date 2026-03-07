# Redis
Redis 사용법은 데이터베이스와 같습니다. 예: `plugin/foo/config/redis.php`
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

사용 시
```php
use support\Redis;
Redis::connection('plugin.foo.default')->get('key');
Redis::connection('plugin.foo.cache')->get('key');
```

같은 방식으로, 메인 프로젝트의 Redis 설정을 재사용하려면
```php
use support\Redis;
Redis::get('key');
// 메인 프로젝트에 cache 연결도 설정되어 있다고 가정
Redis::connection('cache')->get('key');
```
