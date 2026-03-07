# Đóng gói phar

Phar là một loại tệp đóng gói tương tự JAR trong PHP. Bạn có thể sử dụng phar để đóng gói dự án webman của mình thành một tệp phar duy nhất để dễ dàng triển khai.

**Rất cảm ơn [fuzqing](https://github.com/fuzqing) đã đóng góp PR.**

> **Lưu ý**
> Cần tắt tùy chọn cấu hình phar trong `php.ini`, cụ thể là đặt `phar.readonly = 0`.

## Cài đặt công cụ dòng lệnh
`composer require webman/console`

## Đóng gói
Trong thư mục gốc dự án webman, thực hiện lệnh `php webman build:phar`. Một tệp `webman.phar` sẽ được tạo ra trong thư mục `build`.

> Cấu hình đóng gói có trong tệp `config/plugin/webman/console/app.php`.

## Các lệnh bắt đầu và kết thúc
**Bắt đầu**
`php webman.phar start` hoặc `php webman.phar start -d`

**Kết thúc**
`php webman.phar stop`

**Xem trạng thái**
`php webman.phar status`

**Xem trạng thái kết nối**
`php webman.phar connections`

**Khởi động lại**
`php webman.phar restart` hoặc `php webman.phar restart -d`

## Giải thích
* Dự án đã đóng gói không hỗ trợ reload; cần khởi động lại để cập nhật mã nguồn.

* Để tránh kích thước gói quá lớn và tốn bộ nhớ, có thể thiết lập các tùy chọn `exclude_pattern` và `exclude_files` trong `config/plugin/webman/console/app.php` để loại trừ các tệp không cần thiết.

* Sau khi chạy webman.phar, một thư mục `runtime` sẽ được tạo ra ở cùng nơi với webman.phar, để lưu trữ các tệp nhật ký và tạm thời khác.

* Nếu dự án của bạn sử dụng tệp .env, bạn cần đặt tệp .env trong cùng thư mục với webman.phar.

* Tuyệt đối không lưu trữ tệp do người dùng tải lên trong gói phar, vì thao tác với tệp người dùng qua giao thức `phar://` rất nguy hiểm (lỗ hổng deserialization Phar). Tệp do người dùng tải lên phải được lưu trữ riêng trên đĩa ngoài gói phar. Xem bên dưới.

* Nếu doanh nghiệp của bạn cần tải tệp lên thư mục public, bạn cần tách riêng thư mục public và đặt trong cùng thư mục với webman.phar. Lúc này bạn cần thiết lập trong `config/app.php`.
```
'public_path' => base_path(false) . DIRECTORY_SEPARATOR . 'public',
```
Có thể sử dụng hàm trợ giúp `public_path($đường_dẫn_tương_đối)` để tìm vị trí thực tế của thư mục public.

* webman.phar không hỗ trợ chạy các quy trình tùy chỉnh trên Windows.
