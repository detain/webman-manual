# Cấu trúc thư mục

```
plugin/
└── foo
    ├── app
    │   ├── controller
    │   │   └── IndexController.php
    │   ├── exception
    │   │   └── Handler.php
    │   ├── functions.php
    │   ├── middleware
    │   ├── model
    │   └── view
    │       └── index
    │           └── index.html
    ├── config
    │   ├── app.php
    │   ├── autoload.php
    │   ├── container.php
    │   ├── database.php
    │   ├── exception.php
    │   ├── log.php
    │   ├── middleware.php
    │   ├── process.php
    │   ├── redis.php
    │   ├── route.php
    │   ├── static.php
    │   ├── thinkorm.php
    │   ├── translation.php
    │   └── view.php
    ├── public
    └── api
```

Plugin ứng dụng có cùng cấu trúc thư mục và file cấu hình với webman. Thực tế, trải nghiệm phát triển gần như không khác gì so với phát triển ứng dụng webman thông thường.

Thư mục và quy tắc đặt tên plugin tuân theo chuẩn PSR-4. Do các plugin đều đặt trong thư mục `plugin`, nên không gian tên đều bắt đầu bằng `plugin`, ví dụ `plugin\foo\app\controller\UserController`.

## Về thư mục api

Mỗi plugin đều có thư mục `api`. Nếu ứng dụng của bạn cung cấp giao diện nội bộ để ứng dụng khác gọi, hãy đặt các giao diện đó vào thư mục `api`.

Lưu ý: Giao diện ở đây là giao diện gọi hàm, không phải giao diện mạng/HTTP.

Ví dụ: plugin email cung cấp giao diện `Email::send()` tại `plugin/email/api/Email.php` để ứng dụng khác gọi khi gửi email. Ngoài ra, `plugin/email/api/Install.php` được tạo tự động để chợ plugin webman-admin thực hiện cài đặt hoặc gỡ cài đặt.
