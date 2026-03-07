# Cách cài đặt webman

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux: Cài đặt môi trường PHP + Composer (bỏ qua nếu đã có sẵn)
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **Lưu ý**
> Lệnh trên áp dụng cho Linux và macOS. Người dùng Windows cần cài đặt PHP riêng.

Bạn cũng có thể tải xuống thủ công [bản PHP tĩnh](https://www.workerman.net/download) do webman cung cấp và giải nén để sử dụng.

## 1. Tạo dự án

```php
composer create-project workerman/webman:~2.0
```

> **Mẹo**
> Nếu gặp lỗi, có thể bạn đang dùng gương Composer có vấn đề. Chạy `composer config -g --unset repos.packagist` để gỡ proxy.

## 2. Chạy

Đi vào thư mục webman

#### Người dùng Windows
Nhấp đúp vào `windows.bat` hoặc chạy `php windows.php` để khởi động

> **Mẹo**
> Nếu gặp lỗi, có thể một số hàm đã bị vô hiệu hóa. Xem [Kiểm tra hàm bị vô hiệu hóa](others/disable-function-check.md) để gỡ cấm.

#### Người dùng Linux
**Chế độ debug** (cho phát triển: dữ liệu hiển thị ở terminal, dịch vụ webman dừng khi đóng terminal)

```php
php start.php start
```

**Chế độ daemon** (cho môi trường thực tế: dữ liệu không hiển thị ở terminal, dịch vụ webman chạy tiếp sau khi đóng terminal)

```php
php start.php start -d
```

#### Người dùng Docker

Khởi động tất cả dịch vụ và đính kèm vào console
```php
docker-compose up
```

Chạy dịch vụ ở chế độ nền
```php
docker-compose up -d
```

> **Mẹo**
> Nếu gặp lỗi, có thể một số hàm đã bị vô hiệu hóa. Xem [Kiểm tra hàm bị vô hiệu hóa](others/disable-function-check.md) để gỡ cấm.

## 3. Truy cập

Mở trình duyệt truy cập `http://địa-chỉ-ip:8787`.
