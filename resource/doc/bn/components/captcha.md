# ক্যাপচা কম্পোনেন্ট

প্রকল্প লিঙ্ক https://github.com/webman-php/captcha

## ইনস্টলেশন
```
composer require webman/captcha
```

## ব্যবহার

**ফাইল তৈরি করুন `app/controller/LoginController.php`**

```php
<?php
namespace app\controller;

use support\Request;
use Webman\Captcha\CaptchaBuilder;

class LoginController
{
    /**
     * টেস্ট পেজ
     */
    public function index(Request $request)
    {
        return view('login/index');
    }
    
    /**
     * ক্যাপচা ছবি প্রদর্শন করুন
     */
    public function captcha(Request $request)
    {
        // ক্যাপচা ক্লাস ইনিশিয়ালাইজ করুন
        $builder = new CaptchaBuilder;
        // ক্যাপচা তৈরি করুন
        $builder->build();
        // ক্যাপচা মান সেশনে সংরক্ষণ করুন
        $request->session()->set('captcha', strtolower($builder->getPhrase()));
        // ক্যাপচা ছবির বাইনারি ডেটা পেতে
        $img_content = $builder->get();
        // ক্যাপচা বাইনারি ডেটা ফেরত দিন
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }

    /**
     * ক্যাপচা যাচাই করুন
     */
    public function check(Request $request)
    {
        // পোস্ট রিকোয়েস্ট থেকে captcha ফিল্ড পেতে
        $captcha = $request->post('captcha');
        // সেশনের ক্যাপচা মানের সাথে তুলনা করুন
        if (strtolower($captcha) !== $request->session()->get('captcha')) {
            return json(['code' => 400, 'msg' => 'প্রবেশ করা ক্যাপচা সঠিক নয়']);
        }
        return json(['code' => 0, 'msg' => 'ok']);
    }

}
```

**টেমপ্লেট ফাইল তৈরি করুন `app/view/login/index.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>ক্যাপচা টেস্ট</title>  
</head>
<body>
    <form method="post" action="/login/check">
       <img src="/login/captcha" /><br>
        <input type="text" name="captcha" />
        <input type="submit" value="জমা দিন" />
    </form>
</body>
</html>
```

পেজ `http://127.0.0.1:8787/login` এ যান, ইন্টারফেস নিম্নরূপ হবে:
  ![](../../assets/img/captcha.png)

## সাধারণ প্যারামিটার সেটিং
```php
    /**
     * ক্যাপচা ছবি প্রদর্শন করুন
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

আরও ইন্টারফেস এবং প্যারামিটারের জন্য https://github.com/webman-php/captcha দেখুন।
