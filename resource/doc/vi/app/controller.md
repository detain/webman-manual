# Bộ điều khiển

Theo quy tắc PSR4, không gian tên lớp bộ điều khiển bắt đầu bằng `plugin\{plugin_identifier}`, ví dụ

Tạo tệp điều khiển mới `plugin/foo/app/controller/FooController.php`.

```php
<?php
namespace plugin\foo\app\controller;

use support\Request;

class FooController
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

Khi truy cập `http://127.0.0.1:8787/app/foo/foo`, trang sẽ trả về `hello index`

Khi truy cập `http://127.0.0.1:8787/app/foo/foo/hello`, trang sẽ trả về `hello webman`


## Truy cập URL
Đường dẫn địa chỉ URL của plugin ứng dụng đều bắt đầu bằng `/app`, tiếp theo là định danh plugin, rồi đến controller và method cụ thể.
Ví dụ: địa chỉ URL của `plugin\foo\app\controller\UserController` là `http://127.0.0.1:8787/app/foo/user`
