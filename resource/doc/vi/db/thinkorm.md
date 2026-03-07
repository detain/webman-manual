# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) là thành phần cơ sở dữ liệu phát triển dựa trên [top-think/think-orm](https://github.com/top-think/think-orm). Hỗ trợ connection pool và hoạt động trong cả môi trường coroutine và non-coroutine.

## Cài đặt

`composer require -W webman/think-orm`

Sau khi cài đặt cần restart (khởi động lại). reload không có hiệu lực.

## Tệp cấu hình

Chỉnh sửa tệp cấu hình `config/think-orm.php` theo nhu cầu thực tế của bạn.

## Tài liệu

https://www.kancloud.cn/manual/think-orm

## Sử dụng

```php
<?php
namespace app\controller;

use support\Request;
use support\think\Db;

class FooController
{
    public function get(Request $request)
    {
        $user = Db::table('user')->where('uid', '>', 1)->find();
        return json($user);
    }
}
```

## Tạo model

Model think-orm kế thừa từ `support\think\Model`, ví dụ như sau:

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * Bảng liên kết với model.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * Khóa chính của bảng.
     *
     * @var string
     */
    protected $pk = 'id';

}
```

Bạn cũng có thể tạo model think-orm bằng lệnh sau:

```
php webman make:model ten_bang
```

> **Gợi ý**
> Lệnh này yêu cầu cài đặt `webman/console`: `composer require webman/console ^1.2.13`

> **Lưu ý**
> Nếu lệnh make:model phát hiện dự án chính đang dùng `illuminate/database`, nó sẽ tạo file model dựa trên Illuminate thay vì think-orm. Khi đó hãy thêm tham số `tp`: `php webman make:model ten_bang tp` (nếu không hoạt động vui lòng nâng cấp `webman/console`)
