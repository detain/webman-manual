# Giám sát tiến trình
webman đi kèm với một tiến trình giám sát (monitor) tích hợp sẵn, hỗ trợ hai chức năng:
1. Giám sát cập nhật tệp và tự động tải lại mã nghiệp vụ mới (thường dùng trong quá trình phát triển)
2. Giám sát mức tiêu thụ bộ nhớ của tất cả tiến trình; nếu một tiến trình sắp vượt giới hạn `memory_limit` trong `php.ini` thì sẽ tự động khởi động lại tiến trình đó an toàn (không ảnh hưởng nghiệp vụ)

## Cấu hình giám sát
Cấu hình `monitor` trong `config/process.php`:
```php

global $argv;

return [
    // Phát hiện cập nhật tệp và tải lại tự động
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // Giám sát các thư mục này
            'monitorDir' => array_merge([    // Thư mục nào cần giám sát tệp
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Các tệp có phần mở rộng sau sẽ được giám sát
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // Bật giám sát tệp
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // Bật giám sát bộ nhớ
            ]
        ]
    ]
];
```
`monitorDir` dùng để cấu hình giám sát cập nhật ở những thư mục nào (không nên có quá nhiều tệp trong thư mục được giám sát)
`monitorExtensions` dùng để cấu hình phần mở rộng tệp cần giám sát trong các thư mục `monitorDir`
Khi `options.enable_file_monitor` là `true` thì bật giám sát cập nhật tệp (trên Linux bật mặc định khi chạy ở chế độ debug)
Khi `options.enable_memory_monitor` là `true` thì bật giám sát bộ nhớ (không hỗ trợ trên Windows)

> **Mẹo**
> Trên Windows, giám sát cập nhật tệp chỉ được bật khi chạy `windows.bat` hoặc `php windows.php`
