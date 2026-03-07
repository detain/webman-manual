# Captcha-Komponente

Projektadresse https://github.com/webman-php/captcha

## Installation
```
composer require webman/captcha
```

## Verwendung

**Datei `app/controller/LoginController.php` anlegen**

```php
<?php
namespace app\controller;

use support\Request;
use Webman\Captcha\CaptchaBuilder;

class LoginController
{
    /**
     * Testseite
     */
    public function index(Request $request)
    {
        return view('login/index');
    }
    
    /**
     * Captcha-Bild ausgeben
     */
    public function captcha(Request $request)
    {
        // Captcha-Klasse initialisieren
        $builder = new CaptchaBuilder;
        // Captcha generieren
        $builder->build();
        // Captcha-Wert in der Sitzung speichern
        $request->session()->set('captcha', strtolower($builder->getPhrase()));
        // Binärdaten des Captcha-Bildes abrufen
        $img_content = $builder->get();
        // Captcha-Binärdaten ausgeben
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }

    /**
     * Captcha prüfen
     */
    public function check(Request $request)
    {
        // captcha-Feld aus dem POST-Request abrufen
        $captcha = $request->post('captcha');
        // Mit dem Captcha-Wert in der Sitzung vergleichen
        if (strtolower($captcha) !== $request->session()->get('captcha')) {
            return json(['code' => 400, 'msg' => 'Die eingegebene Captcha ist nicht korrekt']);
        }
        return json(['code' => 0, 'msg' => 'ok']);
    }

}
```

**Template-Datei `app/view/login/index.html` anlegen**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Captcha-Test</title>  
</head>
<body>
    <form method="post" action="/login/check">
       <img src="/login/captcha" /><br>
        <input type="text" name="captcha" />
        <input type="submit" value="Absenden" />
    </form>
</body>
</html>
```

Die Seite `http://127.0.0.1:8787/login` aufrufen – die Oberfläche sieht in etwa wie folgt aus:
  ![](../../assets/img/captcha.png)

## Häufige Parameterkonfiguration
```php
    /**
     * Captcha-Bild ausgeben
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

Weitere Schnittstellen und Parameter finden Sie unter https://github.com/webman-php/captcha
