# Thành phần ENV vlucas/phpdotenv

## Giới thiệu
`vlucas/phpdotenv` là thành phần tải biến môi trường, dùng để phân biệt cấu hình giữa các môi trường (như môi trường phát triển, môi trường kiểm thử, v.v.).

## Địa chỉ dự án

https://github.com/vlucas/phpdotenv
  
## Cài đặt
 
```php
composer require vlucas/phpdotenv
 ```
  
## Sử dụng

### Tạo mới file `.env` trong thư mục gốc dự án
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### Chỉnh sửa file cấu hình
**config/database.php**
```php
return [
    // Cơ sở dữ liệu mặc định
    'default' => 'mysql',

    // Cấu hình các cơ sở dữ liệu
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **Gợi ý**
> Nên thêm file `.env` vào danh sách `.gitignore` để tránh đưa vào kho mã nguồn. Thêm file mẫu cấu hình `.env.example` vào kho. Khi triển khai, sao chép `.env.example` thành `.env` rồi chỉnh cấu hình theo môi trường hiện tại. Như vậy dự án sẽ tải cấu hình khác nhau theo từng môi trường.

> **Lưu ý**
> `vlucas/phpdotenv` có thể gặp lỗi với PHP bản TS (Thread Safe). Nên dùng bản NTS (Non-Thread-Safe). Có thể kiểm tra phiên bản PHP hiện tại bằng lệnh `php -v`.

## Thêm thông tin

Truy cập https://github.com/vlucas/phpdotenv
  
