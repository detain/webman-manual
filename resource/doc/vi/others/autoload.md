# Tự động tải

## Tải file theo chuẩn PSR-0 qua Composer
webman tuân theo chuẩn tự động tải `PSR-4`. Nếu dự án của bạn cần tải thư viện tương thích PSR-0, thực hiện các bước sau:

- Tạo thư mục `extend` để chứa thư viện PSR-0
- Chỉnh sửa `composer.json` và thêm nội dung sau vào `autoload`:

```json
"psr-0" : {
    "": "extend/"
}
```
Kết quả cuối cùng sẽ tương tự như sau:
![](../../assets/img/psr0.png)

- Chạy lệnh `composer dumpautoload`
- Chạy lệnh `php start.php restart` để khởi động lại webman (lưu ý: phải khởi động lại mới có hiệu lực)

## Tải một số file qua Composer

- Chỉnh sửa `composer.json` và thêm các file cần tải vào `autoload.files`:
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- Chạy lệnh `composer dumpautoload`
- Chạy lệnh `php start.php restart` để khởi động lại webman (lưu ý: phải khởi động lại mới có hiệu lực)

> **Lưu ý**
> Các file được cấu hình trong `autoload.files` của composer.json sẽ được tải trước khi webman khởi động. Các file được tải qua `config/autoload.php` của framework sẽ được tải sau khi webman khởi động.
> Các file trong `autoload.files` của composer.json khi thay đổi cần restart mới có hiệu lực; reload không có hiệu lực. Các file được tải qua `config/autoload.php` của framework hỗ trợ hot-reload; thay đổi sẽ có hiệu lực sau khi reload.

## Tải một số file qua framework
Một số file có thể không tuân theo chuẩn PSR và không thể tự động tải. Bạn có thể tải chúng bằng cách cấu hình `config/autoload.php`, ví dụ:
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **Lưu ý**
 > Trong `autoload.php` đã thiết lập tải hai file `support/Request.php` và `support/Response.php` vì trong `vendor/workerman/webman-framework/src/support/` cũng có file cùng tên. Qua `autoload.php` chúng ta ưu tiên tải hai file ở thư mục gốc dự án, cho phép tùy chỉnh nội dung mà không cần sửa file trong `vendor`. Nếu bạn không cần tùy chỉnh, có thể bỏ qua hai cấu hình này.
