# Tùy chỉnh 404

Nếu bạn muốn kiểm soát nội dung 404 theo cách động, ví dụ trả về dữ liệu JSON `{"code:"404", "msg":"404 not found"}` khi yêu cầu AJAX và trả về mẫu `app/view/404.html` khi yêu cầu trang, vui lòng tham khảo ví dụ sau.

> Ví dụ dưới đây sử dụng mẫu PHP gốc. Các mẫu khác như `twig`, `blade`, `think-template` cũng áp dụng nguyên tắc tương tự.

**Tạo tệp `app/view/404.html`**
```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>404 not found</title>
</head>
<body>
<?=htmlspecialchars($error)?>
</body>
</html>
```

**Thêm đoạn mã sau vào `config/route.php`:**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // Trả về JSON khi yêu cầu AJAX
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // Trả về mẫu 404.html khi yêu cầu trang
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# Tùy chỉnh 405

Từ webman-framework 1.5.23, callback fallback hỗ trợ tham số `status`. 404 có nghĩa là yêu cầu không tồn tại; 405 có nghĩa là phương thức yêu cầu hiện tại không được hỗ trợ (ví dụ: truy cập bằng GET vào tuyến đường được định nghĩa bằng `Route::post()`).

```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request, $status) {
    $map = [
        404 => '404 not found',
        405 => '405 method not allowed',
    ];
    return response($map[$status], $status);
});
```

# Tùy chỉnh 500

**Tạo `app/view/500.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
Mẫu lỗi tùy chỉnh:
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**Tạo `app/exception/Handler.php`** (tự tạo thư mục nếu chưa tồn tại)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * Hiển thị và trả về phản hồi
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // Trả về dữ liệu JSON khi yêu cầu AJAX
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // Trả về mẫu 500.html khi yêu cầu trang
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**Cấu hình `config/exception.php`:**
```php
return [
    '' => \app\exception\Handler::class,
];
```
