# Thành phần captcha

Địa chỉ dự án https://github.com/webman-php/captcha

## Cài đặt
```
composer require webman/captcha
```

## Sử dụng

**Tạo tệp `app/controller/LoginController.php`**

```php
<?php
namespace app\controller;

use support\Request;
use Webman\Captcha\CaptchaBuilder;

class LoginController
{
    /**
     * Trang kiểm thử
     */
    public function index(Request $request)
    {
        return view('login/index');
    }

    /**
     * Xuất hình ảnh captcha
     */
    public function captcha(Request $request)
    {
        // Khởi tạo lớp captcha
        $builder = new CaptchaBuilder;
        // Tạo captcha
        $builder->build();
        // Lưu giá trị captcha vào phiên
        $request->session()->set('captcha', strtolower($builder->getPhrase()));
        // Lấy dữ liệu nhị phân của hình ảnh captcha
        $img_content = $builder->get();
        // Trả về dữ liệu nhị phân của captcha
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }

    /**
     * Kiểm tra captcha
     */
    public function check(Request $request)
    {
        // Lấy trường captcha từ yêu cầu POST
        $captcha = $request->post('captcha');
        // So sánh với giá trị captcha trong phiên
        if (strtolower($captcha) !== $request->session()->get('captcha')) {
            return json(['code' => 400, 'msg' => 'Mã captcha nhập vào không đúng']);
        }
        return json(['code' => 0, 'msg' => 'ok']);
    }
}
```

**Tạo tệp mẫu `app/view/login/index.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Kiểm tra captcha</title>
</head>
<body>
    <form method="post" action="/login/check">
       <img src="/login/captcha" /><br>
        <input type="text" name="captcha" />
        <input type="submit" value="Gửi" />
    </form>
</body>
</html>
```

Truy cập trang `http://127.0.0.1:8787/login`, giao diện sẽ tương tự như sau:
  ![](../../assets/img/captcha.png)

## Cài đặt thông số phổ biến
```php
    /**
     * Xuất hình ảnh captcha
     */
    public function captcha(Request $request)
    {
        $builder = new PhraseBuilder(4, 'abcdefghjkmnpqrstuvwxyzABCDEFGHJKMNPQRSTUVWXYZ');
        $captcha = new CaptchaBuilder(null, $builder);
        $captcha->build();
        $request->session()->set('join', strtolower($captcha->getPhrase()));
        $img_content = $captcha->get();
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }
```

Xem thêm về các giao diện và thông số tại: https://github.com/webman-php/captcha
