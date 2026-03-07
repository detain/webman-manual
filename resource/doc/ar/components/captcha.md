# مكون التحقق من الصورة

رابط المشروع https://github.com/webman-php/captcha

## التثبيت
```
composer require webman/captcha
```

## الاستخدام

**إنشاء ملف `app/controller/LoginController.php`**

```php
<?php
namespace app\controller;

use support\Request;
use Webman\Captcha\CaptchaBuilder;

class LoginController
{
    /**
     * صفحة الاختبار
     */
    public function index(Request $request)
    {
        return view('login/index');
    }
    
    /**
     * عرض صورة التحقق
     */
    public function captcha(Request $request)
    {
        // تهيئة فئة التحقق
        $builder = new CaptchaBuilder;
        // إنشاء التحقق
        $builder->build();
        // تخزين قيمة التحقق في الجلسة
        $request->session()->set('captcha', strtolower($builder->getPhrase()));
        // الحصول على البيانات الثنائية لصورة التحقق
        $img_content = $builder->get();
        // إرجاع البيانات الثنائية للتحقق
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }

    /**
     * التحقق من صحة الإدخال
     */
    public function check(Request $request)
    {
        // الحصول على حقل captcha من طلب POST
        $captcha = $request->post('captcha');
        // المقارنة مع قيمة التحقق في الجلسة
        if (strtolower($captcha) !== $request->session()->get('captcha')) {
            return json(['code' => 400, 'msg' => 'رمز التحقق المدخل غير صحيح']);
        }
        return json(['code' => 0, 'msg' => 'ok']);
    }

}
```

**إنشاء ملف القالب `app/view/login/index.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>اختبار التحقق</title>  
</head>
<body>
    <form method="post" action="/login/check">
       <img src="/login/captcha" /><br>
        <input type="text" name="captcha" />
        <input type="submit" value="إرسال" />
    </form>
</body>
</html>
```

بعد الدخول إلى الصفحة `http://127.0.0.1:8787/login` ستظهر واجهة مشابهة لما يلي:
  ![](../../assets/img/captcha.png)

## الإعدادات الشائعة للمعلمات
```php
    /**
     * عرض صورة التحقق
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

لمزيد من الواجهات والمعلمات، راجع https://github.com/webman-php/captcha
