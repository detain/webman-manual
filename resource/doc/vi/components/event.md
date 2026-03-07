# Xử lý sự kiện
`webman/event` cung cấp cơ chế sự kiện tinh tế, cho phép thực thi logic kinh doanh mà không cần sửa đổi mã nguồn, đạt được sự tách biệt giữa các module. Ví dụ điển hình: khi người dùng mới đăng ký thành công, chỉ cần phát ra sự kiện tùy chỉnh như `user.register`, và mỗi module có thể nhận sự kiện này để thực thi logic tương ứng.

## Cài đặt
`composer require webman/event`

## Đăng ký sự kiện
Đăng ký sự kiện được cấu hình tập trung qua tệp `config/event.php`.

```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...các hàm xử lý sự kiện khác...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...các hàm xử lý sự kiện khác...
    ]
];
```

**Lưu ý:**
- `user.register`, `user.logout`, v.v. là tên sự kiện (kiểu chuỗi). Nên dùng chữ thường và phân tách bằng dấu chấm (`.`).
- Một sự kiện có thể có nhiều hàm xử lý, gọi theo thứ tự cấu hình.

## Hàm xử lý sự kiện
Hàm xử lý có thể là phương thức lớp, hàm hoặc closure.

Ví dụ: tạo lớp `app/event/User.php` (tạo thư mục nếu chưa có).

```php
<?php
namespace app\event;
class User
{
    function register($user)
    {
        var_export($user);
    }
 
    function logout($user)
    {
        var_export($user);
    }
}
```

## Phát sự kiện
Dùng `Event::dispatch($event_name, $data);` hoặc `Event::emit($event_name, $data);` để phát sự kiện. Ví dụ:

```php
<?php
namespace app\controller;
use support\Request;
use Webman\Event\Event;
class User
{
    public function register(Request $request)
    {
        $user = [
            'name' => 'webman',
            'age' => 2
        ];
        Event::dispatch('user.register', $user);
    }
}
```

Có hai hàm để phát sự kiện: `Event::dispatch($event_name, $data);` và `Event::emit($event_name, $data);` — tham số giống nhau. Khác biệt: `emit` tự bắt ngoại lệ; nếu một hàm gây lỗi, các hàm khác vẫn chạy. Còn `dispatch` không bắt ngoại lệ; nếu hàm nào gây lỗi thì dừng và ngoại lệ được truyền lên.

> **Gợi ý**
> Tham số `$data` có thể là bất kỳ dữ liệu nào (mảng, thể hiện lớp, chuỗi, v.v.).

## Lắng nghe sự kiện bằng ký tự đại diện
Đăng ký bằng ký tự đại diện cho phép xử lý nhiều sự kiện bằng cùng một trình nghe. Ví dụ trong `config/event.php`:

```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```

Có thể lấy tên sự kiện cụ thể qua tham số thứ hai `$event_data` của hàm xử lý:

```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // tên sự kiện cụ thể, như user.register, user.logout, v.v.
        var_export($user);
    }
}
```

## Dừng phát sự kiện
Khi hàm xử lý trả về `false`, việc phát sự kiện đó sẽ dừng lại.

## Xử lý sự kiện bằng closure
Hàm xử lý có thể là phương thức lớp hoặc closure. Ví dụ:

```php
<?php
return [
    'user.login' => [
        function($user){
            var_dump($user);
        }
    ]
];
```

## Xem sự kiện và trình nghe
Dùng lệnh `php webman event:list` để xem tất cả sự kiện và trình nghe đã cấu hình trong dự án.

## Phạm vi hỗ trợ
Ngoài dự án chính, [plugin cơ sở](../plugin/base.md) và [plugin ứng dụng](../app/app.md) cũng hỗ trợ cấu hình `event.php`.
**Tệp cấu hình plugin cơ sở:** `config/plugin/nhà-cung-cấp/tên-plugin/event.php`
**Tệp cấu hình plugin ứng dụng:** `plugin/tên-plugin/config/event.php`

## Lưu ý
Xử lý sự kiện không phải bất đồng bộ và không phù hợp cho nghiệp vụ chậm; nghiệp vụ chậm nên dùng hàng đợi thông báo, ví dụ [webman/redis-queue](https://www.workerman.net/plugin/12).
