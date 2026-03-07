# Nhật ký
Cách sử dụng lớp nhật ký tương tự như cách sử dụng cơ sở dữ liệu
```php
use support\Log;
Log::channel('plugin.admin.default')->info('test');
```

Nếu muốn tái sử dụng cấu hình nhật ký của dự án chính, sử dụng trực tiếp như sau:
```php
use support\Log;
Log::info('Nội dung nhật ký');
// Giả sử dự án chính có cấu hình nhật ký tên test
Log::channel('test')->info('Nội dung nhật ký');
```
