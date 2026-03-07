# Tệp cấu hình

Cấu hình plugin hoạt động giống như trong dự án webman thông thường, nhưng thường chỉ áp dụng cho plugin hiện tại và không ảnh hưởng đến dự án chính.
Ví dụ, giá trị của `plugin.foo.app.controller_suffix` chỉ ảnh hưởng đến hậu tố điều khiển của plugin, không ảnh hưởng đến dự án chính.
Ví dụ, giá trị của `plugin.foo.app.controller_reuse` chỉ ảnh hưởng đến việc plugin có tái sử dụng điều khiển hay không, không ảnh hưởng đến dự án chính.
Ví dụ, giá trị của `plugin.foo.middleware` chỉ ảnh hưởng đến middleware của plugin, không ảnh hưởng đến dự án chính.
Ví dụ, giá trị của `plugin.foo.view` chỉ ảnh hưởng đến chế độ xem mà plugin sử dụng, không ảnh hưởng đến dự án chính.
Ví dụ, giá trị của `plugin.foo.container` chỉ ảnh hưởng đến container mà plugin sử dụng, không ảnh hưởng đến dự án chính.
Ví dụ, giá trị của `plugin.foo.exception` chỉ ảnh hưởng đến lớp xử lý ngoại lệ của plugin, không ảnh hưởng đến dự án chính.

Tuy nhiên, vì định tuyến là toàn cục nên các tuyến đường do plugin cấu hình cũng ảnh hưởng đến định tuyến toàn cục.

## Lấy cấu hình
Để lấy cấu hình của một plugin, sử dụng `config('plugin.{plugin}.{cấu_hình_cụ_thể}');`. Ví dụ, để lấy toàn bộ cấu hình của `plugin/foo/config/app.php`, sử dụng `config('plugin.foo.app')`.
Tương tự, dự án chính hoặc các plugin khác có thể dùng `config('plugin.foo.xxx')` để lấy cấu hình của plugin foo.

## Cấu hình không được hỗ trợ
Plugin ứng dụng không hỗ trợ cấu hình `server.php` và `session.php`, cũng không hỗ trợ cấu hình `app.request_class`, `app.public_path` và `app.runtime_path`.
