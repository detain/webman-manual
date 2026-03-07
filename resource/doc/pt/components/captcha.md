# Componente de captcha

Endereço do projeto https://github.com/webman-php/captcha

## Instalação
```
composer require webman/captcha
```

## Uso

**Criar o arquivo `app/controller/LoginController.php`**

```php
<?php
namespace app\controller;

use support\Request;
use Webman\Captcha\CaptchaBuilder;

class LoginController
{
    /**
     * Página de teste
     */
    public function index(Request $request)
    {
        return view('login/index');
    }
    
    /**
     * Gerar imagem do captcha
     */
    public function captcha(Request $request)
    {
        // Inicializar a classe de captcha
        $builder = new CaptchaBuilder;
        // Gerar o captcha
        $builder->build();
        // Armazenar o valor do captcha na sessão
        $request->session()->set('captcha', strtolower($builder->getPhrase()));
        // Obter os dados binários da imagem do captcha
        $img_content = $builder->get();
        // Retornar os dados binários do captcha
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }

    /**
     * Verificar o captcha
     */
    public function check(Request $request)
    {
        // Obter o campo captcha da requisição POST
        $captcha = $request->post('captcha');
        // Comparar com o valor do captcha na sessão
        if (strtolower($captcha) !== $request->session()->get('captcha')) {
            return json(['code' => 400, 'msg' => 'O captcha inserido está incorreto']);
        }
        return json(['code' => 0, 'msg' => 'ok']);
    }

}
```

**Criar o arquivo de modelo `app/view/login/index.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Teste de captcha</title>  
</head>
<body>
    <form method="post" action="/login/check">
       <img src="/login/captcha" /><br>
        <input type="text" name="captcha" />
        <input type="submit" value="Enviar" />
    </form>
</body>
</html>
```

Acesse a página `http://127.0.0.1:8787/login`, a interface será semelhante à seguinte:
  ![](../../assets/img/captcha.png)

## Configurações comuns de parâmetros
```php
    /**
     * Gerar imagem do captcha
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

Para mais interfaces e parâmetros, consulte https://github.com/webman-php/captcha
