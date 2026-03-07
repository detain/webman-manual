# Quy trình thực thi

## Quy trình khởi động tiến trình

Khi thực thi `php start.php start`, quy trình thực thi như sau:

1. Tải cấu hình dưới config/
2. Thiết lập cấu hình liên quan đến Worker như `pid_file`, `stdout_file`, `log_file`, `max_package_size`, v.v.
3. Tạo tiến trình webman và lắng nghe cổng (mặc định: 8787)
4. Tạo các tiến trình tùy chỉnh dựa trên cấu hình
5. Sau khi tiến trình webman và các tiến trình tùy chỉnh được khởi động, thực hiện logic sau (tất cả trong `onWorkerStart`):
   ① Tải các tệp được cấu hình trong `config/autoload.php`, như `app/functions.php`
   ② Tải các middleware được cấu hình trong `config/middleware.php` (bao gồm `config/plugin/*/*/middleware.php`)
   ③ Thực thi phương thức `start` của các lớp được cấu hình trong `config/bootstrap.php` (bao gồm `config/plugin/*/*/bootstrap.php`) để khởi tạo các module, như kết nối cơ sở dữ liệu Laravel
   ④ Tải các route được định nghĩa trong `config/route.php` (bao gồm `config/plugin/*/*/route.php`)

## Quy trình xử lý yêu cầu

1. Kiểm tra xem URL yêu cầu có tương ứng với tệp tĩnh dưới public không. Nếu có thì trả về tệp (kết thúc yêu cầu). Nếu không thì chuyển sang bước 2.
2. Xác định xem URL có khớp với route nào không. Nếu không khớp thì chuyển sang bước 3; nếu khớp thì chuyển sang bước 4.
3. Kiểm tra xem route mặc định có bị tắt không. Nếu có thì trả về 404 (kết thúc yêu cầu). Nếu không thì chuyển sang bước 4.
4. Tìm các middleware của controller tương ứng với yêu cầu, thực hiện thao tác tiền xử lý của middleware theo thứ tự (giai đoạn yêu cầu của mô hình củ hành), thực hiện logic nghiệp vụ của controller, thực hiện thao tác hậu xử lý của middleware (giai đoạn phản hồi của mô hình củ hành), kết thúc yêu cầu. (Xem [Mô hình củ hành của Middleware](https://www.workerman.net/doc/webman/middleware.html#%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%B4%8B%E8%91%B1%E6%A8%A1%E5%9E%8B))
