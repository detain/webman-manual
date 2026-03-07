# Cache

[webman/cache](https://github.com/webman-php/cache) là thành phần cache dựa trên [symfony/cache](https://github.com/symfony/cache), tương thích với môi trường coroutine và phi coroutine, hỗ trợ connection pool.

## Cài đặt

```php
composer require -W webman/cache
```

## Ví dụ
```php
<?php
namespace app\controller;

use support\Request;
use support\Cache;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Cache::set($key, rand());
        return response(Cache::get($key));
    }
}
```

## Vị trí file cấu hình
File cấu hình nằm tại `config/cache.php`. Tạo thủ công nếu chưa có.

## Nội dung file cấu hình
```php
<?php
return [
    'default' => 'file',
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default'
        ],
        'array' => [
            'driver' => 'array'
        ],
        'apcu' => [
            'driver' => 'apcu'
        ]
    ]
];
```
`stores.driver` hỗ trợ bốn driver: **file**, **redis**, **array** và **apcu**.

#### Driver file
Driver mặc định. Không phụ thuộc thành phần ngoài. Hỗ trợ chia sẻ cache giữa các tiến trình. Không hỗ trợ chia sẻ giữa nhiều máy chủ.

#### Driver array
Lưu trữ trong bộ nhớ, hiệu năng tốt nhất nhưng tiêu tốn bộ nhớ. Không hỗ trợ chia sẻ giữa các tiến trình hoặc máy chủ. Dữ liệu mất khi tiến trình khởi động lại. Thường dùng cho dự án có khối lượng cache nhỏ.

#### Driver apcu
Lưu trữ trong bộ nhớ. Hiệu năng chỉ sau array. Hỗ trợ chia sẻ cache giữa các tiến trình. Không hỗ trợ chia sẻ giữa nhiều máy chủ. Dữ liệu mất khi tiến trình khởi động lại. Thường dùng cho dự án có khối lượng cache nhỏ.

> Cần cài đặt và bật [tiện ích mở rộng APCu](https://pecl.php.net/package/APCu); không khuyến khích dùng cho các tình huống ghi/xóa cache thường xuyên, sẽ làm giảm hiệu năng rõ rệt.

#### Driver redis
Phụ thuộc thành phần [webman/redis](./redis.md). Hỗ trợ chia sẻ cache giữa các tiến trình và máy chủ.

**stores.redis.connection**

`stores.redis.connection` tương ứng với key được định nghĩa trong `config/redis.php`. Khi dùng Redis sẽ tái sử dụng cấu hình `webman/redis`, bao gồm cấu hình connection pool.

**Nên thêm cấu hình Redis riêng cho cache trong `config/redis.php`, ví dụ:**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== Thêm mới
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

Sau đó đặt `stores.redis.connection` là `cache`. File `config/cache.php` cuối cùng tương tự như sau:

```php
<?php
return [
    'default' => 'redis', // <====
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache' // <====
        ],
        'array' => [
            'driver' => 'array'
        ]
    ]
];
```

## Chuyển đổi kho lưu trữ
Có thể chuyển đổi kho lưu trữ thủ công để dùng các driver khác nhau, ví dụ:

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **Mẹo**
> Tên key cache bị giới hạn theo [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) và không được chứa bất kỳ ký tự nào trong `{}()/\@:`. Từ phiên bản `symfony/cache` 7.2.4 có thể tạm thời bỏ qua kiểm tra này bằng cấu hình PHP ini `zend.assertions=-1`.

## Sử dụng thành phần Cache khác

Xem [cơ sở dữ liệu khác](others.md#ThinkCache) để sử dụng thành phần [ThinkCache](https://github.com/webman-php/think-cache).
