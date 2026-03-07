# Proxy nginx

Khi webman cần cung cấp truy cập trực tiếp từ mạng bên ngoài, khuyến nghị thêm proxy nginx phía trước webman. Điều này mang lại những lợi ích sau:

- Tài nguyên tĩnh được xử lý bởi nginx, giúp webman tập trung vào logic nghiệp vụ
- Nhiều phiên bản webman có thể chia sẻ cổng 80 và 443, phân biệt các trang web theo tên miền, cho phép nhiều trang trên một máy chủ
- Cho phép kiến trúc php-fpm và webman cùng tồn tại
- Proxy nginx với SSL cho https đơn giản và hiệu quả hơn
- Có thể lọc nghiêm ngặt các yêu cầu không hợp lệ từ mạng bên ngoài

## Ví dụ proxy nginx
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name ten_mien_trang;
  listen 80;
  access_log off;
  # Quan trọng: root phải trỏ đến thư mục public trong webman, không phải thư mục gốc của webman
  root /your/webman/public;

  location / {
    try_files $uri @proxy;
  }

  location @proxy {
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_pass http://webman;
  }

  # Từ chối truy cập tất cả tệp có đuôi .php
  location ~ \.php$ {
      return 404;
  }

  # Cho phép truy cập thư mục .well-known
  location ~ ^/\.well-known/ {
    allow all;
  }

  # Từ chối truy cập tất cả tệp hoặc thư mục bắt đầu bằng .
  location ~ /\. {
      return 404;
  }

}
```

Thông thường, nhà phát triển chỉ cần cấu hình server_name và root với giá trị thực; các trường khác không cần cấu hình.

> **Lưu ý**
> Đặc biệt quan trọng: tùy chọn root phải trỏ đến thư mục public trong webman. Không bao giờ đặt thành thư mục gốc của webman, nếu không tất cả tệp của bạn có thể bị tải xuống và truy cập từ internet, bao gồm cả tệp nhạy cảm như cấu hình cơ sở dữ liệu.
