# Personalizar 404

Se você quiser controlar dinamicamente o conteúdo do 404, por exemplo retornar dados JSON `{"code:"404", "msg":"404 not found"}` em requisições AJAX e retornar o template `app/view/404.html` em requisições de página, consulte o exemplo a seguir.

> O exemplo usa templates nativos de PHP. Outros templates como `twig`, `blade`, `think-template` seguem o mesmo princípio.

**Criar o arquivo `app/view/404.html`**
```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>404 not found</title>
</head>
<body>
<?=htmlspecialchars($error)?>
</body>
</html>
```

**Adicionar o seguinte código em `config/route.php`:**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // Retornar JSON em requisições AJAX
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // Retornar o template 404.html em requisições de página
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# Personalizar 405

A partir do webman-framework 1.5.23, o callback de fallback suporta o parâmetro `status`. 404 significa que a requisição não existe; 405 significa que o método de requisição atual não é permitido (ex.: acessar via GET uma rota definida com `Route::post()`).

```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request, $status) {
    $map = [
        404 => '404 not found',
        405 => '405 method not allowed',
    ];
    return response($map[$status], $status);
});
```

# Personalizar 500

**Criar `app/view/500.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
Template de erro personalizado:
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**Criar `app/exception/Handler.php`** (criar o diretório se não existir)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * Renderizar e retornar a resposta
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // Retornar dados JSON em requisições AJAX
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // Retornar o template 500.html em requisições de página
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**Configurar `config/exception.php`**
```php
return [
    '' => \app\exception\Handler::class,
];
```
