# Đóng gói

Ví dụ: đóng gói plugin ứng dụng foo

* Đặt số phiên bản trong `plugin/foo/config/app.php` (**Quan trọng**)
* Xóa các tệp không cần thiết trong `plugin/foo`, đặc biệt là các tệp tạm thời dùng để kiểm tra chức năng tải lên trong `plugin/foo/public`
* Nếu dự án của bạn bao gồm tạo bảng cơ sở dữ liệu và các thao tác tương tự, hãy cấu hình đúng `plugin/foo/install.sql`. Xem [phần cơ sở dữ liệu](database.md)
* Nếu dự án của bạn có cấu hình cơ sở dữ liệu và Redis riêng biệt, hãy xóa các cấu hình này trước. Các cấu hình này cần được kích hoạt qua trình hướng dẫn cài đặt khi truy cập ứng dụng lần đầu (bạn cần tự triển khai), để quản trị viên nhập và tạo thủ công.
* Nếu dự án của bạn có menu backend webman admin, hãy cấu hình `plugin/foo/config/menu.php` để các menu này được thiết lập tự động khi cài đặt plugin. Xem [webman-admin nhập menu](https://www.workerman.net/doc/webman-admin/app-development/menu.html)
* Khôi phục các tệp khác cần được trả về trạng thái ban đầu
* Sau khi hoàn thành các bước trên, vào thư mục `{dự án chính}/plugin/`
* Người dùng Linux: chạy lệnh `zip -r foo.zip foo` để tạo foo.zip
* Người dùng Windows: nhấp chuột phải vào thư mục foo và chọn "Nén thành tệp ZIP" để tạo foo.zip

**foo.zip là tệp đã đóng gói. Tham khảo chương tiếp theo [Xuất bản plugin](publish.md)**
