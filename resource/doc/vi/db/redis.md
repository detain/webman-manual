# Redis

[webman/redis](https://github.com/webman-php/redis) mở rộng [illuminate/redis](https://github.com/illuminate/redis) với pool kết nối, hỗ trợ môi trường có và không có coroutine, cách dùng giống Laravel.

Bạn phải cài đặt extension redis cho `php-cli` trước khi dùng `illuminate/redis`.

## Cài đặt

```php
composer require -W webman/redis illuminate/events
```

Sau khi cài đặt cần restart (reload không có tác dụng).

## Cấu hình

Tệp cấu hình Redis nằm trong `config/redis.php`:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // Cấu hình pool kết nối
            'max_connections' => 10,     // Số kết nối tối đa trong pool
            'min_connections' => 1,      // Số kết nối tối thiểu trong pool
            'wait_timeout' => 3,         // Thời gian chờ tối đa khi lấy kết nối (giây)
            'idle_timeout' => 50,        // Hết thời gian chờ, kết nối được giải phóng cho đến min_connections
            'heartbeat_interval' => 50,  // Chu kỳ heartbeat (không vượt quá 60 giây)
        ],
    ]
];
```

## Về pool kết nối

* Mỗi tiến trình có pool riêng; pool không chia sẻ giữa các tiến trình.
* Khi không dùng coroutine, thực thi tuần tự nên tối đa chỉ dùng một kết nối.
* Khi dùng coroutine, thực thi song song và pool điều chỉnh từ `min_connections` đến `max_connections`.
* Khi số coroutine dùng Redis vượt quá `max_connections`, chúng chờ tối đa `wait_timeout` giây; sau đó ném ngoại lệ.
* Khi rảnh (có hoặc không coroutine), kết nối được giải phóng sau `idle_timeout` cho đến khi còn `min_connections` (cho phép 0).

## Ví dụ

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

## API Redis

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

Tương đương với:

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

> **Lưu ý**
> Cẩn thận khi dùng `Redis::select($db)`. Vì webman là framework thường trú trong bộ nhớ, nếu một request dùng `Redis::select($db)` đổi cơ sở dữ liệu thì sẽ ảnh hưởng đến các request sau. Với nhiều cơ sở dữ liệu, nên cấu hình mỗi `$db` là một kết nối Redis riêng.

## Dùng nhiều kết nối Redis

Ví dụ trong tệp cấu hình `config/redis.php`:

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

Mặc định dùng kết nối cấu hình trong `default`. Dùng `Redis::connection()` để chọn kết nối Redis cần dùng:

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## Cấu hình cluster

Nếu ứng dụng dùng cluster Redis, định nghĩa trong cấu hình với khóa `clusters`:

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

Mặc định cluster có thể sharding phía client trên các nút, cho phép pool nút và nhiều bộ nhớ. Lưu ý sharding phía client không xử lý lỗi; phù hợp chủ yếu cho dữ liệu cache từ cơ sở dữ liệu chính khác. Để dùng cluster Redis gốc, khai báo trong `options` của cấu hình:

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

## Lệnh pipeline

Khi cần gửi nhiều lệnh trong một thao tác, nên dùng pipeline. Phương thức `pipeline` nhận một closure; tất cả lệnh được thực thi trong một lần:

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
