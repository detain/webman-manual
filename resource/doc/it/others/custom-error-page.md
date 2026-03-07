# Personalizzare 404

Se si desidera controllare dinamicamente il contenuto del 404, ad esempio restituire dati JSON `{"code:"404", "msg":"404 not found"}` per le richieste AJAX e restituire il template `app/view/404.html` per le richieste di pagina, fare riferimento all'esempio seguente.

> L'esempio utilizza template PHP nativi. Altri template come `twig`, `blade`, `think-template` seguono lo stesso principio.

**Creare il file `app/view/404.html`**
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

**Aggiungere il seguente codice in `config/route.php`:**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // Restituire JSON per le richieste AJAX
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // Restituire il template 404.html per le richieste di pagina
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# Personalizzare 405

Dalla versione webman-framework 1.5.23, il callback di fallback supporta il parametro `status`. 404 indica che la richiesta non esiste; 405 indica che il metodo di richiesta attuale non è supportato (es. accedere con GET a una route definita con `Route::post()`).

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

# Personalizzare 500

**Creare `app/view/500.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
Template di errore personalizzato:
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**Creare `app/exception/Handler.php`** (creare la cartella se non esiste)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * Renderizzare e restituire la risposta
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // Restituire dati JSON per le richieste AJAX
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // Restituire il template 500.html per le richieste di pagina
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**Configurare `config/exception.php`**
```php
return [
    '' => \app\exception\Handler::class,
];
```
