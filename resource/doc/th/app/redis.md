# Redis
การใช้ Redis คล้ายกับการใช้ฐานข้อมูล เช่น `plugin/foo/config/redis.php`
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
เมื่อใช้งาน
```php
use support\Redis;
Redis::connection('plugin.foo.default')->get('key');
Redis::connection('plugin.foo.cache')->get('key');
```

ในทำนองเดียวกัน หากต้องการใช้การตั้งค่า Redis ของโปรเจกต์หลักซ้ำ
```php
use support\Redis;
Redis::get('key');
// สมมติว่าโปรเจกต์หลักมีการตั้งค่าการเชื่อมต่อ cache ไว้แล้ว
Redis::connection('cache')->get('key');
```
