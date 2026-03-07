# คอมโพเนนต์รหัสยืนยันตัวตน

ที่อยู่โปรเจกต์ https://github.com/webman-php/captcha

## การติดตั้ง
```
composer require webman/captcha
```

## การใช้งาน

**สร้างไฟล์ `app/controller/LoginController.php`**

```php
<?php
namespace app\controller;

use support\Request;
use Webman\Captcha\CaptchaBuilder;

class LoginController
{
    /**
     * หน้าทดสอบ
     */
    public function index(Request $request)
    {
        return view('login/index');
    }
    
    /**
     * แสดงรูปภาพรหัสยืนยันตัวตน
     */
    public function captcha(Request $request)
    {
        // เริ่มต้นคลาสรหัสยืนยันตัวตน
        $builder = new CaptchaBuilder;
        // สร้างรหัสยืนยันตัวตน
        $builder->build();
        // จัดเก็บค่ารหัสยืนยันตัวตนในเซสชัน
        $request->session()->set('captcha', strtolower($builder->getPhrase()));
        // รับข้อมูลไบนารีของรูปภาพรหัสยืนยันตัวตน
        $img_content = $builder->get();
        // คืนค่าข้อมูลไบนารีของรหัสยืนยันตัวตน
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }

    /**
     * ตรวจสอบรหัสยืนยันตัวตน
     */
    public function check(Request $request)
    {
        // รับฟิลด์ captcha จากคำขอ POST
        $captcha = $request->post('captcha');
        // เปรียบเทียบกับค่ารหัสยืนยันตัวตนในเซสชัน
        if (strtolower($captcha) !== $request->session()->get('captcha')) {
            return json(['code' => 400, 'msg' => 'รหัสยืนยันตัวตนที่ป้อนไม่ถูกต้อง']);
        }
        return json(['code' => 0, 'msg' => 'ok']);
    }

}
```

**สร้างไฟล์เทมเพลต `app/view/login/index.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>ทดสอบรหัสยืนยันตัวตน</title>  
</head>
<body>
    <form method="post" action="/login/check">
       <img src="/login/captcha" /><br>
        <input type="text" name="captcha" />
        <input type="submit" value="ส่ง" />
    </form>
</body>
</html>
```

เข้าสู่หน้า `http://127.0.0.1:8787/login` หน้าตาจะคล้ายกับดังนี้:
  ![](../../assets/img/captcha.png)

## การตั้งค่าพารามิเตอร์ที่ใช้บ่อย
```php
    /**
     * แสดงรูปภาพรหัสยืนยันตัวตน
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

ดูข้อมูลเพิ่มเติมเกี่ยวกับอินเตอร์เฟซและพารามิเตอร์ได้ที่ https://github.com/webman-php/captcha
