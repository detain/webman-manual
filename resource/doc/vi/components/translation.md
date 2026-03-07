# Đa ngôn ngữ

Hỗ trợ đa ngôn ngữ sử dụng component [symfony/translation](https://github.com/symfony/translation).

## Cài đặt
```
composer require symfony/translation
```

## Tạo gói ngôn ngữ
webman mặc định lưu gói ngôn ngữ trong thư mục `resource/translations` (tạo mới nếu chưa có). Để đổi thư mục, cấu hình trong `config/translation.php`.
Mỗi ngôn ngữ tương ứng với một thư mục con, định nghĩa ngôn ngữ mặc định đặt trong `messages.php`. Ví dụ:
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

Tất cả file ngôn ngữ trả về một mảng, ví dụ:
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## Cấu hình

`config/translation.php`

```php
return [
    // Ngôn ngữ mặc định
    'locale' => 'zh_CN',
    // Ngôn ngữ dự phòng: khi không tìm thấy bản dịch trong ngôn ngữ hiện tại sẽ thử dùng bản dịch của ngôn ngữ dự phòng
    'fallback_locale' => ['zh_CN', 'en'],
    // Thư mục lưu file ngôn ngữ
    'path' => base_path() . '/resource/translations',
];
```

## Dịch

Dùng phương thức `trans()` để dịch.

Tạo file ngôn ngữ `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 世界!',
];
```

Tạo file `app/controller/UserController.php`:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        $hello = trans('hello'); // 你好 世界!
        return response($hello);
    }
}
```

Truy cập `http://127.0.0.1:8787/user/get` sẽ trả về "你好 世界!"

## Đổi ngôn ngữ mặc định

Dùng phương thức `locale()` để chuyển ngôn ngữ.

Thêm file ngôn ngữ `resource/translations/en/messages.php`:
```php
return [
    'hello' => 'hello world!',
];
```

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // Chuyển ngôn ngữ
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
Truy cập `http://127.0.0.1:8787/user/get` sẽ trả về "hello world!"

Bạn cũng có thể dùng tham số thứ 4 của hàm `trans()` để tạm thời chuyển ngôn ngữ. Ví dụ trên tương đương với:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // Tham số thứ 4 chuyển ngôn ngữ
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## Thiết lập ngôn ngữ rõ ràng cho từng request
translation là singleton, nghĩa là mọi request dùng chung một instance. Nếu request nào dùng `locale()` để đặt ngôn ngữ mặc định sẽ ảnh hưởng đến mọi request sau đó trong process. Vì vậy cần thiết lập ngôn ngữ rõ ràng cho từng request. Ví dụ dùng middleware sau:

Tạo file `app/middleware/Lang.php` (tạo thư mục nếu chưa có):
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Lang implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        locale(session('lang', 'zh_CN'));
        return $handler($request);
    }
}
```

Thêm middleware toàn cục trong `config/middleware.php`:
```php
return [
    // Middleware toàn cục
    '' => [
        // ... các middleware khác bỏ qua
        app\middleware\Lang::class,
    ]
];
```


## Dùng placeholder
Đôi khi một thông báo chứa biến cần dịch, ví dụ
```php
trans('hello ' . $name);
```
Trường hợp này dùng placeholder để xử lý.

Cập nhật `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 %name%!',
];
```
Khi dịch, truyền giá trị tương ứng với placeholder qua tham số thứ 2:
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## Xử lý số nhiều
Một số ngôn ngữ cấu trúc câu khác nhau theo số lượng. Ví dụ `There is %count% apple` đúng khi `%count%` là 1, sai khi lớn hơn 1.

Trường hợp này dùng **dấu pipe** (`|`) để liệt kê dạng số nhiều.

Thêm `apple_count` vào file `resource/translations/en/messages.php`:
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

Có thể chỉ định khoảng số để tạo quy tắc số nhiều phức tạp hơn:
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## Chỉ định file ngôn ngữ

Tên file ngôn ngữ mặc định là `messages.php`, thực tế có thể tạo file ngôn ngữ tên khác.

Tạo file ngôn ngữ `resource/translations/zh_CN/admin.php`:
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

Chỉ định file ngôn ngữ qua tham số thứ 3 của `trans()` (bỏ phần mở rộng `.php`).
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## Thêm thông tin
Xem [tài liệu symfony/translation](https://symfony.com/doc/current/translation.html)
