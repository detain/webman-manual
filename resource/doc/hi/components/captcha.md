# कैप्चा कंपोनेंट

प्रोजेक्ट लिंक https://github.com/webman-php/captcha

## स्थापना
```
composer require webman/captcha
```

## उपयोग

**फ़ाइल बनाएं `app/controller/LoginController.php`**

```php
<?php
namespace app\controller;

use support\Request;
use Webman\Captcha\CaptchaBuilder;

class LoginController
{
    /**
     * परीक्षण पृष्ठ
     */
    public function index(Request $request)
    {
        return view('login/index');
    }
    
    /**
     * कैप्चा छवि आउटपुट करें
     */
    public function captcha(Request $request)
    {
        // कैप्चा कक्षा को प्रारंभ करें
        $builder = new CaptchaBuilder;
        // कैप्चा उत्पन्न करें
        $builder->build();
        // कैप्चा मान को सत्र में संग्रहीत करें
        $request->session()->set('captcha', strtolower($builder->getPhrase()));
        // कैप्चा छवि का बाइनरी डेटा प्राप्त करें
        $img_content = $builder->get();
        // कैप्चा बाइनरी डेटा आउटपुट करें
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }

    /**
     * कैप्चा जाँचें
     */
    public function check(Request $request)
    {
        // पोस्ट अनुरोध से captcha फ़ील्ड प्राप्त करें
        $captcha = $request->post('captcha');
        // सत्र में कैप्चा मान से तुलना करें
        if (strtolower($captcha) !== $request->session()->get('captcha')) {
            return json(['code' => 400, 'msg' => 'दर्ज किया गया कैप्चा गलत है']);
        }
        return json(['code' => 0, 'msg' => 'ok']);
    }

}
```

**टेम्पलेट फ़ाइल बनाएं `app/view/login/index.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>कैप्चा परीक्षण</title>  
</head>
<body>
    <form method="post" action="/login/check">
       <img src="/login/captcha" /><br>
        <input type="text" name="captcha" />
        <input type="submit" value="जमा करें" />
    </form>
</body>
</html>
```

पृष्ठ `http://127.0.0.1:8787/login` पर जाएं, इंटरफ़ेस निम्नलिखित के समान दिखेगा:
  ![](../../assets/img/captcha.png)

## सामान्य पैरामीटर सेटिंग्स
```php
    /**
     * कैप्चा छवि आउटपुट करें
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

अधिक इंटरफ़ेस और पैरामीटर के लिए https://github.com/webman-php/captcha देखें।
