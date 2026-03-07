# Hướng dẫn nhanh cơ sở dữ liệu (Dựa trên thành phần Laravel)

[webman/database](https://github.com/webman-php/database) được xây dựng trên [illuminate/database](https://github.com/illuminate/database) và bổ sung connection pool cho môi trường coroutine và phi coroutine. Cách dùng giống Laravel.

Bạn cũng có thể tham khảo [Sử dụng thành phần cơ sở dữ liệu khác](others.md) để dùng ThinkPHP hoặc cơ sở dữ liệu khác.

## Cài đặt cơ sở dữ liệu

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

Sau khi cài đặt cần khởi động lại bằng restart (reload không có tác dụng).

> **Gợi ý**
> webman/database phụ thuộc vào `illuminate/database` của Laravel, nên các gói dependency sẽ được cài tự động.

> **Lưu ý**
> Nếu không cần phân trang, sự kiện cơ sở dữ liệu hoặc ghi log SQL, chỉ cần chạy:
> `composer require -W webman/database`

## Cấu hình cơ sở dữ liệu
`config/database.php`
```php

return [
    // Cơ sở dữ liệu mặc định
    'default' => 'mysql',

    // Cấu hình kết nối
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'test',
            'username'    => 'root',
            'password'    => '',
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
            'options' => [
                PDO::ATTR_EMULATE_PREPARES => false, // Bắt buộc khi dùng swoole hoặc swow
            ],
            'pool' => [ // Cấu hình connection pool
                'max_connections' => 5, // Số kết nối tối đa
                'min_connections' => 1, // Số kết nối tối thiểu
                'wait_timeout' => 3,    // Thời gian chờ tối đa khi lấy kết nối từ pool; vượt sẽ ném exception. Chỉ có hiệu lực trong môi trường coroutine
                'idle_timeout' => 60,   // Thời gian chờ tối đa của kết nối trong pool; vượt sẽ đóng và thu hồi đến khi còn min_connections
                'heartbeat_interval' => 50, // Khoảng thời gian kiểm tra nhịp của pool (giây); nên đặt dưới 60
            ],
        ],
    ],
];
```

Ngoài cấu hình `pool`, các mục khác giống Laravel.

## Về connection pool
* Mỗi tiến trình có pool riêng; pool không chia sẻ giữa các tiến trình.
* Khi không dùng coroutine, các request chạy tuần tự trong tiến trình, không có đồng thời, nên pool tối đa chỉ có một kết nối.
* Khi dùng coroutine, các request chạy đồng thời trong tiến trình; pool điều chỉnh số kết nối theo nhu cầu, không vượt `max_connections`, không thấp hơn `min_connections`.
* Vì pool tối đa `max_connections`, khi số coroutine truy cập cơ sở dữ liệu vượt mức này, một số sẽ xếp hàng chờ tối đa `wait_timeout` giây; vượt quá sẽ ném exception.
* Khi nhàn rỗi (cả coroutine và phi coroutine), kết nối sẽ được thu hồi sau `idle_timeout` cho đến khi còn `min_connections` (`min_connections` có thể là 0).


## Ví dụ sử dụng cơ sở dữ liệu
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        $default_uid = 29;
        $uid = $request->get('uid', $default_uid);
        $name = Db::table('users')->where('uid', $uid)->value('username');
        return response("hello $name");
    }
}
```

Cách dùng giống Laravel: dùng phương thức `Db::table()` để thao tác với cơ sở dữ liệu.
