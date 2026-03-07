# Componente captcha

Indirizzo del progetto https://github.com/webman-php/captcha

## Installazione
```
composer require webman/captcha
```

## Utilizzo

**Creare il file `app/controller/LoginController.php`**

```php
<?php
namespace app\controller;

use support\Request;
use Webman\Captcha\CaptchaBuilder;

class LoginController
{
    /**
     * Pagina di prova
     */
    public function index(Request $request)
    {
        return view('login/index');
    }
    
    /**
     * Generare l'immagine del captcha
     */
    public function captcha(Request $request)
    {
        // Inizializzare la classe del captcha
        $builder = new CaptchaBuilder;
        // Generare il captcha
        $builder->build();
        // Salvare il valore del captcha nella sessione
        $request->session()->set('captcha', strtolower($builder->getPhrase()));
        // Ottenere i dati binari dell'immagine del captcha
        $img_content = $builder->get();
        // Restituire i dati binari del captcha
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }

    /**
     * Verificare il captcha
     */
    public function check(Request $request)
    {
        // Ottenere il campo captcha dalla richiesta POST
        $captcha = $request->post('captcha');
        // Confrontare con il valore del captcha nella sessione
        if (strtolower($captcha) !== $request->session()->get('captcha')) {
            return json(['code' => 400, 'msg' => 'Il captcha inserito non è corretto']);
        }
        return json(['code' => 0, 'msg' => 'ok']);
    }

}
```

**Creare il file del modello `app/view/login/index.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Test del captcha</title>  
</head>
<body>
    <form method="post" action="/login/check">
       <img src="/login/captcha" /><br>
        <input type="text" name="captcha" />
        <input type="submit" value="Invia" />
    </form>
</body>
</html>
```

Accedere alla pagina `http://127.0.0.1:8787/login`, l'aspetto sarà simile al seguente:
  ![](../../assets/img/captcha.png)

## Impostazioni dei parametri comuni
```php
    /**
     * Generare l'immagine del captcha
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

Per ulteriori interfacce e parametri, fare riferimento a https://github.com/webman-php/captcha
