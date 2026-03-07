# Cấu trúc thư mục
```
.
├── app                           Thư mục ứng dụng
│   ├── controller                Thư mục điều khiển
│   ├── model                     Thư mục mô hình
│   ├── view                      Thư mục giao diện
│   ├── middleware                Thư mục middleware
│   │   └── StaticFile.php        Middleware tệp tĩnh tích hợp sẵn
│   ├── process                   Thư mục tiến trình tùy chỉnh
│   │   ├── Http.php              Tiến trình Http
│   │   └── Monitor.php           Tiến trình giám sát
│   └── functions.php             Các hàm tùy chỉnh nghiệp vụ viết vào tệp này
├── config                        Thư mục cấu hình
│   ├── app.php                   Cấu hình ứng dụng
│   ├── autoload.php              Các tệp cấu hình tại đây sẽ được tự động tải
│   ├── bootstrap.php             Cấu hình callback chạy khi tiến trình khởi động (onWorkerStart)
│   ├── container.php             Cấu hình container
│   ├── dependence.php            Cấu hình phụ thuộc của container
│   ├── database.php              Cấu hình cơ sở dữ liệu
│   ├── exception.php             Cấu hình ngoại lệ
│   ├── log.php                   Cấu hình nhật ký
│   ├── middleware.php            Cấu hình middleware
│   ├── process.php               Cấu hình tiến trình tùy chỉnh
│   ├── redis.php                 Cấu hình Redis
│   ├── route.php                 Cấu hình định tuyến
│   ├── server.php                Cấu hình máy chủ (cổng, số tiến trình, v.v.)
│   ├── view.php                  Cấu hình giao diện
│   ├── static.php                Cấu hình tệp tĩnh và middleware tệp tĩnh
│   ├── translation.php           Cấu hình đa ngôn ngữ
│   └── session.php               Cấu hình phiên
├── public                        Thư mục tài nguyên tĩnh
├── runtime                       Thư mục thời gian chạy của ứng dụng, cần quyền ghi
├── start.php                     Tệp khởi động dịch vụ
├── vendor                        Thư mục thư viện bên thứ ba cài đặt bởi Composer
└── support                       Điều chỉnh thư viện (bao gồm thư viện bên thứ ba)
    ├── Request.php               Lớp yêu cầu
    ├── Response.php              Lớp phản hồi
    ├── Setup.php                 Tệp kịch bản hướng dẫn cài đặt
    └── bootstrap.php             Tệp kịch bản khởi tạo sau khi tiến trình khởi động
```
