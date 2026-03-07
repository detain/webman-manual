# Plugin cơ bản

Plugin cơ bản thường là các thành phần dùng chung, thường được cài đặt qua Composer, mã nguồn đặt trong thư mục vendor. Khi cài đặt, có thể tự động sao chép các cấu hình tùy chỉnh (middleware, process, route, v.v.) vào thư mục `{dự án chính}config/plugin`. webman sẽ tự động nhận diện cấu hình trong thư mục này và hợp nhất vào cấu hình chính, qua đó cho phép plugin can thiệp vào bất kỳ giai đoạn nào trong vòng đời của webman.

Để biết thêm, xem [Tạo plugin cơ bản](create.md).
