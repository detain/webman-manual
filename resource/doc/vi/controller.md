# Bộ điều khiển

Tạo tệp điều khiển mới `app/controller/FooController.php`.

```php
<?php
namespace app\controller;

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

Khi truy cập `http://127.0.0.1:8787/foo`, trang sẽ trả về `hello index`.

Khi truy cập `http://127.0.0.1:8787/foo/hello`, trang sẽ trả về `hello webman`.

Bạn có thể thay đổi quy tắc định tuyến qua cấu hình định tuyến, xem [Định tuyến](route.md).

> **Gợi ý**
> Nếu gặp lỗi 404, hãy mở `config/app.php`, đặt `controller_suffix` thành `Controller` và khởi động lại.

## Hậu tố điều khiển
Từ phiên bản 1.3, webman hỗ trợ thiết lập hậu tố điều khiển trong `config/app.php`. Nếu `controller_suffix` trong `config/app.php` được đặt là chuỗi rỗng `''`, điều khiển sẽ có dạng:

`app\controller\Foo.php`.

```php
<?php
namespace app\controller;

use support\Request;

class Foo
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

Nên đặt hậu tố điều khiển là `Controller` để tránh xung đột tên với lớp mô hình và tăng bảo mật.

## Giải thích
- Framework tự động truyền đối tượng `support\Request` vào điều khiển, qua đó có thể lấy dữ liệu đầu vào (get, post, header, cookie, v.v.), xem [Yêu cầu](request.md).
- Điều khiển có thể trả về số, chuỗi hoặc đối tượng `support\Response`, nhưng không thể trả về kiểu dữ liệu khác.
- Đối tượng `support\Response` có thể tạo qua các hàm trợ giúp như `response()`, `json()`, `xml()`, `jsonp()`, `redirect()`, v.v.

## Ràng buộc tham số điều khiển

#### Ví dụ
webman hỗ trợ ràng buộc tự động tham số yêu cầu với tham số phương thức điều khiển. Ví dụ:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

Bạn có thể truyền giá trị `name` và `age` qua `GET` hoặc `POST`, hoặc qua tham số đường dẫn. Ví dụ:

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

Thứ tự ưu tiên: `tham số đường dẫn` > `GET` > `POST`.

#### Giá trị mặc định

Khi truy cập `/user/create?name=tom`, sẽ gặp lỗi:

```html
Missing input parameter age
```

Do không truyền tham số `age`. Có thể xử lý bằng cách đặt giá trị mặc định. Ví dụ:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age = 18): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

#### Kiểu tham số
Khi truy cập `/user/create?name=tom&age=not_int`, sẽ gặp lỗi:

> **Gợi ý**
> Để tiện kiểm thử, chúng ta truyền tham số trực tiếp trong thanh địa chỉ. Trong phát triển thực tế nên truyền qua `POST`.

```html
Input age must be of type int, string given
```

Dữ liệu nhận được sẽ được chuyển theo kiểu. Nếu không chuyển được sẽ ném ngoại lệ `support\exception\InputTypeException`. Vì `age` không chuyển được sang `int` nên gặp lỗi trên.

#### Thông báo lỗi tùy chỉnh
Có thể tùy chỉnh thông báo như `Missing input parameter age` và `Input age must be of type int, string given` qua đa ngôn ngữ. Tham khảo lệnh:

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => 'Tham số :parameter phải có kiểu :exceptType, kiểu truyền vào là :actualType',
    'Missing input parameter :parameter' => 'Thiếu tham số :parameter',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### Các kiểu khác
webman hỗ trợ kiểu `int`, `float`, `string`, `bool`, `array`, `object`, `thể hiện lớp`, v.v. Ví dụ:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age, float $balance, bool $vip, array $extension): Response
    {
        return json([
            'name' => $name,
            'age' => $age,
            'balance' => $balance,
            'vip' => $vip,
            'extension' => $extension,
        ]);
    }
}
```

Khi truy cập `/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar`, sẽ nhận được:

```json
{
  "name": "tom",
  "age": 18,
  "balance": 100.5,
  "vip": true,
  "extension": {
    "foo": "bar"
  }
}
```

#### Thể hiện lớp
webman hỗ trợ truyền thể hiện lớp qua type hint. Ví dụ:

**app\service\Blog.php**
```php
<?php
namespace app\service;
class Blog
{
    private $title;
    private $content;
    public function __construct(string $title, string $content)
    {
        $this->title = $title;
        $this->content = $content;
    }
    public function get()
    {
        return [
            'title' => $this->title,
            'content' => $this->content,
        ];
    }
}
```

**app\controller\BlogController.php**
```php
<?php
namespace app\controller;
use app\service\Blog;
use support\Response;

class BlogController
{
    public function create(Blog $blog): Response
    {
        return json($blog->get());
    }
}
```

Khi truy cập `/blog/create?blog[title]=hello&blog[content]=world`, sẽ nhận được:

```json
{
  "title": "hello",
  "content": "world"
}
```

#### Thể hiện mô hình

**app\model\User.php**
```php
<?php
namespace app\model;
use support\Model;
class User extends Model
{
    protected $connection = 'mysql';
    protected $table = 'user';
    protected $primaryKey = 'id';
    public $timestamps = false;
    // Cần thêm các trường có thể điền ở đây để tránh trường không an toàn từ giao diện
    protected $fillable = ['name', 'age'];
}
```

**app\controller\UserController.php**
```php
<?php
namespace app\controller;
use app\model\User;
class UserController
{
    public function create(User $user): int
    {
        $user->save();
        return $user->id;
    }
}
```

Khi truy cập `/user/create?user[name]=tom&user[age]=18`, sẽ nhận được kết quả tương tự:

```json
1
```

## Vòng đời điều khiển

Khi `controller_reuse` trong `config/app.php` là `false`, mỗi yêu cầu khởi tạo một thể hiện điều khiển tương ứng, sau khi yêu cầu kết thúc thể hiện bị hủy. Giống cơ chế framework truyền thống.

Khi `controller_reuse` là `true`, mọi yêu cầu dùng chung thể hiện điều khiển. Tức là thể hiện khi tạo sẽ nằm trong bộ nhớ và được tái sử dụng.

> **Lưu ý**
> Khi bật tái sử dụng điều khiển, yêu cầu không nên thay đổi thuộc tính của điều khiển vì sẽ ảnh hưởng yêu cầu tiếp theo. Ví dụ:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    protected $model;
    
    public function update(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->update();
        return response('ok');
    }
    
    public function delete(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->delete();
        return response('ok');
    }
    
    protected function getModel($id)
    {
        // Phương thức này sẽ giữ model sau yêu cầu đầu update?id=1
        // Nếu yêu cầu delete?id=2, sẽ xóa dữ liệu id=1
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **Gợi ý**
> Return dữ liệu trong constructor `__construct()` của điều khiển không có hiệu lực. Ví dụ:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // Return trong constructor không có hiệu lực, trình duyệt không nhận phản hồi này
        return response('hello'); 
    }
}
```

## Khác biệt giữa không tái sử dụng và tái sử dụng điều khiển

#### Không tái sử dụng
Mỗi yêu cầu đều tạo thể hiện điều khiển mới, kết thúc yêu cầu thì giải phóng và thu hồi bộ nhớ. Giống framework truyền thống, phù hợp thói quen đa số lập trình viên. Do tạo và hủy lặp lại nên hiệu năng kém hơn tái sử dụng một chút (benchmark helloworld chênh khoảng 10%, với nghiệp vụ thực tế gần như bỏ qua được).

#### Tái sử dụng
Mỗi tiến trình chỉ new điều khiển một lần, kết thúc yêu cầu không giải phóng thể hiện. Yêu cầu tiếp theo trong tiến trình dùng lại thể hiện đó. Hiệu năng tốt hơn nhưng không hợp thói quen đa số lập trình viên.

#### Trường hợp không dùng tái sử dụng điều khiển

Khi yêu cầu thay đổi thuộc tính điều khiển thì không nên bật tái sử dụng vì thay đổi đó ảnh hưởng yêu cầu tiếp theo.

Một số lập trình viên thích khởi tạo cho từng yêu cầu trong constructor `__construct()` của điều khiển. Khi đó không thể dùng tái sử dụng vì constructor của tiến trình chỉ gọi một lần, không phải mỗi yêu cầu đều gọi.
