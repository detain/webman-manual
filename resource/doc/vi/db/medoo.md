# Cơ sở dữ liệu Medoo

[webman/medoo](https://github.com/webman-php/medoo) mở rộng [Medoo](https://medoo.in/) với hỗ trợ connection pool và hoạt động trong cả môi trường coroutine và không coroutine. Cách sử dụng giống như Medoo.

## Cài đặt
`composer require webman/medoo`

## Cấu hình cơ sở dữ liệu Medoo
Vị trí tệp cấu hình: `config/plugin/webman/medoo/database.php`

## Sử dụng cơ sở dữ liệu Medoo
```php
<?php
namespace app\controller;

use support\Request;
use support\Medoo;

class Index
{
    public function index(Request $request)
    {
        $user = Medoo::get('user', '*', ['uid' => 1]);
        return json($user);
    }
}
```

> **Gợi ý**
> `Medoo::get('user', '*', ['uid' => 1]);`
> tương đương với
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## Cấu hình nhiều cơ sở dữ liệu Medoo

**Cấu hình**
Thêm cấu hình mới trong `config/plugin/webman/medoo/database.php` với khóa bất kỳ; ở đây sử dụng `other`.

```php
<?php
return [
    'default' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [ // Cấu hình connection pool
            'max_connections' => 5, // Số kết nối tối đa
            'min_connections' => 1, // Số kết nối tối thiểu
            'wait_timeout' => 60,   // Thời gian chờ tối đa khi lấy kết nối từ pool; ném ngoại lệ nếu vượt quá
            'idle_timeout' => 3,    // Thời gian nhàn rỗi tối đa của kết nối trong pool; vượt quá sẽ đóng đến min_connections
            'heartbeat_interval' => 50, // Khoảng heartbeat của pool (giây); khuyến nghị dưới 60 giây
        ]
    ],
    // Thêm cấu hình 'other' tại đây
    'other' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [
            'max_connections' => 5,
            'min_connections' => 1,
            'wait_timeout' => 60,
            'idle_timeout' => 3,
            'heartbeat_interval' => 50,
        ],
    ],
];
```

## Sử dụng cơ sở dữ liệu Medoo
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

Xem [tài liệu chính thức Medoo](https://medoo.in/api/select)
