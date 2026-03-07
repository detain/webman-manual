# Redis
Cách dùng Redis tương tự cơ sở dữ liệu, ví dụ `plugin/foo/config/redis.php`
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
Khi dùng
```php
use support\Redis;
Redis::connection('plugin.foo.default')->get('key');
Redis::connection('plugin.foo.cache')->get('key');
```

Tương tự, nếu muốn dùng lại cấu hình Redis của dự án chính
```php
use support\Redis;
Redis::get('key');
// Giả sử dự án chính đã cấu hình thêm kết nối cache
Redis::connection('cache')->get('key');
```
