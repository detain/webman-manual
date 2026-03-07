# Tệp Cấu Hình

## Vị Trí
Các tệp cấu hình webman nằm trong thư mục `config/`. Bạn có thể sử dụng hàm `config()` để truy cập các cấu hình tương ứng trong dự án của mình.

## Truy Cập Cấu Hình

Lấy tất cả cấu hình:
```php
config();
```

Lấy tất cả cấu hình trong `config/app.php`:
```php
config('app');
```

Lấy cấu hình `debug` trong `config/app.php`:
```php
config('app.debug');
```

Nếu cấu hình là mảng, bạn có thể sử dụng `.` để truy cập các giá trị lồng nhau. Ví dụ:
```php
config('file.key1.key2');
```

## Giá Trị Mặc Định
```php
config($key, $default);
```
Truyền giá trị mặc định làm tham số thứ hai. Nếu cấu hình không tồn tại, giá trị mặc định sẽ được trả về. Nếu cấu hình không tồn tại và không có mặc định được đặt, trả về `null`.

## Cấu Hình Tùy Chỉnh
Nhà phát triển có thể thêm tệp cấu hình riêng vào thư mục `config/`. Ví dụ:

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**Cách sử dụng khi truy cập cấu hình**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## Thay Đổi Cấu Hình
Webman không hỗ trợ thay đổi cấu hình động. Tất cả cấu hình phải được thay đổi thủ công trong các tệp cấu hình tương ứng, sau đó tải lại hoặc khởi động lại ứng dụng.

> **Lưu ý**
> Cấu hình máy chủ `config/server.php` và cấu hình tiến trình `config/process.php` không hỗ trợ tải lại. Bạn phải khởi động lại để thay đổi có hiệu lực.

## Lưu Ý Quan Trọng
Nếu bạn tạo tệp cấu hình trong thư mục con dưới `config/`, ví dụ `config/order/status.php`, bạn cần có tệp `app.php` trong thư mục `config/order/` với nội dung sau:
```php
<?php
return [
    'enable' => true,
];
```
`enable` đặt thành `true` cho khung biết cần tải cấu hình từ thư mục này.
Cấu trúc thư mục cấu hình phải như sau:
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
Sau đó bạn có thể truy cập mảng hoặc các khóa cụ thể từ `status.php` qua `config.order.status`.


## Tham Chiếu Tệp Cấu Hình

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // Cổng lắng nghe (đã loại bỏ từ 1.6.0, cấu hình trong config/process.php)
    'transport' => 'tcp', // Giao thức truyền tải (đã loại bỏ từ 1.6.0, cấu hình trong config/process.php)
    'context' => [], // SSL v.v. (đã loại bỏ từ 1.6.0, cấu hình trong config/process.php)
    'name' => 'webman', // Tên tiến trình (đã loại bỏ từ 1.6.0, cấu hình trong config/process.php)
    'count' => cpu_count() * 4, // Số tiến trình (đã loại bỏ từ 1.6.0, cấu hình trong config/process.php)
    'user' => '', // Người dùng (đã loại bỏ từ 1.6.0, cấu hình trong config/process.php)
    'group' => '', // Nhóm (đã loại bỏ từ 1.6.0, cấu hình trong config/process.php)
    'reusePort' => false, // Bật tái sử dụng cổng (đã loại bỏ từ 1.6.0, cấu hình trong config/process.php)
    'event_loop' => '',  // Lớp vòng lặp sự kiện, mặc định tự động chọn
    'stop_timeout' => 2, // Thời gian chờ tối đa khi nhận tín hiệu dừng/khởi động lại/tải lại, ép thoát nếu tiến trình không thoát đúng thời gian
    'pid_file' => runtime_path() . '/webman.pid', // Vị trí tệp PID
    'status_file' => runtime_path() . '/webman.status', // Vị trí tệp trạng thái
    'stdout_file' => runtime_path() . '/logs/stdout.log', // Vị trí tệp stdout, mọi đầu ra sau khi webman khởi động đều được ghi vào đây
    'log_file' => runtime_path() . '/logs/workerman.log', // Vị trí tệp nhật ký Workerman
    'max_package_size' => 10 * 1024 * 1024 // Kích thước gói tối đa, 10M. Kích thước tệp tải lên bị giới hạn bởi giá trị này
];
```

#### app.php
```php
return [
    'debug' => true,  // Chế độ gỡ lỗi, bật stack trace v.v. khi có lỗi. Nên tắt trong môi trường sản xuất
    'error_reporting' => E_ALL, // Mức độ báo lỗi
    'default_timezone' => 'Asia/Shanghai', // Múi giờ mặc định
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // Đường dẫn thư mục công khai
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // Đường dẫn thư mục runtime
    'controller_suffix' => 'Controller', // Hậu tố controller
    'controller_reuse' => false, // Có tái sử dụng controller hay không
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // Cấu hình tiến trình webman
    'webman' => [ 
        'handler' => Http::class, // Lớp xử lý tiến trình
        'listen' => 'http://0.0.0.0:8787', // Địa chỉ lắng nghe
        'count' => cpu_count() * 4, // Số tiến trình, mặc định 4x CPU
        'user' => '', // Người dùng tiến trình, nên dùng người dùng quyền thấp
        'group' => '', // Nhóm tiến trình, nên dùng nhóm quyền thấp
        'reusePort' => false, // Bật reusePort, phân phối kết nối qua các tiến trình worker
        'eventLoop' => '', // Lớp vòng lặp sự kiện, dùng server.event_loop khi trống
        'context' => [], // Ngữ cảnh lắng nghe, v.d. SSL
        'constructor' => [ // Tham số constructor cho trình xử lý tiến trình, ở đây là lớp Http
            'requestClass' => Request::class, // Lớp yêu cầu tùy chỉnh
            'logger' => Log::channel('default'), // Phiên bản logger
            'appPath' => app_path(), // Đường dẫn thư mục ứng dụng
            'publicPath' => public_path() // Đường dẫn thư mục công khai
        ]
    ],
    // Tiến trình giám sát tự động tải lại khi thay đổi tệp và phát hiện rò rỉ bộ nhớ
    'monitor' => [
        'handler' => app\process\Monitor::class, // Lớp xử lý
        'reloadable' => false, // Tiến trình này không thực hiện reload
        'constructor' => [ // Tham số constructor cho trình xử lý tiến trình
            // Các thư mục cần theo dõi, tránh quá nhiều vì làm chậm phát hiện
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Phần mở rộng tệp cần theo dõi thay đổi
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // Các tùy chọn khác
            'options' => [
                // Bật giám sát tệp, chỉ Linux, giám sát tệp mặc định tắt ở chế độ daemon
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // Bật giám sát bộ nhớ, chỉ Linux
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// Trả về phiên bản container tiêm phụ thuộc PSR-11
return new Webman\Container;
```

#### dependence.php
```php
// Cấu hình dịch vụ và phụ thuộc trong container tiêm phụ thuộc
return [];
```

#### route.php
```php

use support\Route;
// Định nghĩa route cho đường dẫn /test
Route::any('/test', function (Request $request) {
    return response('test');
});
```

#### view.php
```php
use support\view\Raw;
use support\view\Twig;
use support\view\Blade;
use support\view\ThinkPHP;

return [
    'handler' => Raw::class // Lớp xử lý view mặc định
];
```

### autoload.php
```php
// Cấu hình tệp autoload của framework
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php',
        base_path() . '/support/Response.php',
    ]
];
```

#### cache.php
```php
// Cấu hình cache
return [
    'default' => 'file', // Driver mặc định
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // Đường dẫn lưu trữ tệp cache
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // Tên kết nối Redis, tham chiếu cấu hình trong redis.php
        ],
        'array' => [
            'driver' => 'array' // Cache trong bộ nhớ, xóa khi khởi động lại
        ]
    ]
];
```

#### redis.php
```php
return [
    'default' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 0,
    ],
];
```

#### database.php
```php
return [
 // Cơ sở dữ liệu mặc định
 'default' => 'mysql',
 // Cấu hình kết nối cơ sở dữ liệu
 'connections' => [

     'mysql' => [
         'driver'      => 'mysql',
         'host'        => '127.0.0.1',
         'port'        => 3306,
         'database'    => 'webman',
         'username'    => 'webman',
         'password'    => '',
         'unix_socket' => '',
         'charset'     => 'utf8',
         'collation'   => 'utf8_unicode_ci',
         'prefix'      => '',
         'strict'      => true,
         'engine'      => null,
     ],

     'sqlite' => [
         'driver'   => 'sqlite',
         'database' => '',
         'prefix'   => '',
     ],

     'pgsql' => [
         'driver'   => 'pgsql',
         'host'     => '127.0.0.1',
         'port'     => 5432,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
         'schema'   => 'public',
         'sslmode'  => 'prefer',
     ],

     'sqlsrv' => [
         'driver'   => 'sqlsrv',
         'host'     => 'localhost',
         'port'     => 1433,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
     ],
 ],
];
```

#### exception.php
```php
return [
    // Đặt lớp xử lý ngoại lệ
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // Handler
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // Tên tệp nhật ký
                    7, //$maxFiles // Giữ nhật ký 7 ngày
                    Monolog\Logger::DEBUG, // Mức nhật ký
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // Formatter
                    'constructor' => [null, 'Y-m-d H:i:s', true], // Tham số formatter
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // Loại
    'type' => 'file', // hoặc redis hoặc redis_cluster
     // Handler
    'handler' => FileSessionHandler::class,
     // Cấu hình
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // Thư mục lưu trữ
        ],
        'redis' => [
            'host' => '127.0.0.1',
            'port' => 6379,
            'auth' => '',
            'timeout' => 2,
            'database' => '',
            'prefix' => 'redis_session_',
        ],
        'redis_cluster' => [
            'host' => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth' => '',
            'prefix' => 'redis_session_',
        ]
    ],
    'session_name' => 'PHPSID', // Tên phiên
    'auto_update_timestamp' => false, // Tự động cập nhật timestamp để tránh hết hạn phiên
    'lifetime' => 7*24*60*60, // Thời gian tồn tại
    'cookie_lifetime' => 365*24*60*60, // Thời gian tồn tại cookie
    'cookie_path' => '/', // Đường dẫn cookie
    'domain' => '', // Tên miền cookie
    'http_only' => true, // Chỉ HTTP
    'secure' => false, // Chỉ HTTPS
    'same_site' => '', // Thuộc tính SameSite
    'gc_probability' => [1, 1000], // Xác suất thu gom rác phiên
];
```

#### middleware.php
```php
// Cấu hình middleware
return [];
```

#### static.php
```php
return [
    'enable' => true, // Bật phục vụ tệp tĩnh webman
    'middleware' => [ // Middleware tệp tĩnh, cho chính sách cache, CORS v.v.
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // Ngôn ngữ mặc định
    'locale' => 'zh_CN',
    // Ngôn ngữ dự phòng
    'fallback_locale' => ['zh_CN', 'en'],
    // Vị trí tệp dịch
    'path' => base_path() . '/resource/translations',
];
```
