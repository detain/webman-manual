# Captcha bileşeni

Proje adresi https://github.com/webman-php/captcha

## Kurulum
```
composer require webman/captcha
```

## Kullanım

**Dosya oluştur `app/controller/LoginController.php`**

```php
<?php
namespace app\controller;

use support\Request;
use Webman\Captcha\CaptchaBuilder;

class LoginController
{
    /**
     * Test sayfası
     */
    public function index(Request $request)
    {
        return view('login/index');
    }
    
    /**
     * Captcha görüntüsü oluştur
     */
    public function captcha(Request $request)
    {
        // Captcha sınıfını başlat
        $builder = new CaptchaBuilder;
        // Captcha oluştur
        $builder->build();
        // Captcha değerini oturuma kaydet
        $request->session()->set('captcha', strtolower($builder->getPhrase()));
        // Captcha görüntüsünün ikili verilerini al
        $img_content = $builder->get();
        // Captcha ikili verilerini döndür
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }

    /**
     * Captcha'yı kontrol et
     */
    public function check(Request $request)
    {
        // POST isteğindeki captcha alanını al
        $captcha = $request->post('captcha');
        // Oturumdaki captcha değeriyle karşılaştır
        if (strtolower($captcha) !== $request->session()->get('captcha')) {
            return json(['code' => 400, 'msg' => 'Girilen captcha doğru değil']);
        }
        return json(['code' => 0, 'msg' => 'ok']);
    }

}
```

**Şablon dosyası oluştur `app/view/login/index.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Captcha Testi</title>  
</head>
<body>
    <form method="post" action="/login/check">
       <img src="/login/captcha" /><br>
        <input type="text" name="captcha" />
        <input type="submit" value="Gönder" />
    </form>
</body>
</html>
```

`http://127.0.0.1:8787/login` sayfasına gidildiğinde arayüz şu şekilde görünecektir:
  ![](../../assets/img/captcha.png)

## Sık kullanılan parametre ayarları
```php
    /**
     * Captcha görüntüsü oluştur
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

Daha fazla arayüz ve parametre için https://github.com/webman-php/captcha adresine bakabilirsiniz.
