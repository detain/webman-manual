# Cài đặt

Có hai cách cài đặt plugin ứng dụng:

## Cài đặt từ chợ plugin
Vào [trang quản trị webman-admin chính thức](https://www.workerman.net/plugin/82), mở trang plugin ứng dụng rồi nhấn nút cài đặt để cài plugin tương ứng.

## Cài đặt từ gói mã nguồn
Tải file nén plugin ứng dụng từ chợ ứng dụng, giải nén và tải thư mục đã giải nén lên `{dự án chính}/plugin/` (nếu thư mục plugin chưa có thì tạo thủ công), sau đó chạy `php webman app-plugin:install tên_plugin` để hoàn tất cài đặt.

Ví dụ: Nếu tên file nén là ai.zip, giải nén vào `{dự án chính}/plugin/ai`, sau đó chạy `php webman app-plugin:install ai` để hoàn tất cài đặt.

# Gỡ cài đặt

Tương tự, có hai cách gỡ cài đặt plugin ứng dụng:

## Gỡ cài đặt từ chợ plugin
Vào [trang quản trị webman-admin chính thức](https://www.workerman.net/plugin/82), mở trang plugin ứng dụng rồi nhấn nút gỡ cài đặt để gỡ plugin tương ứng.

## Gỡ cài đặt từ gói mã nguồn
Chạy `php webman app-plugin:uninstall tên_plugin` để gỡ cài đặt, sau đó xóa thủ công thư mục plugin tương ứng trong `{dự án chính}/plugin/`.
