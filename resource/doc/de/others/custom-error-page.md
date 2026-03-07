# Benutzerdefinierte 404

Wenn Sie den Inhalt von 404 dynamisch steuern möchten, z.B. bei AJAX-Anfragen JSON-Daten `{"code:"404", "msg":"404 not found"}` zurückgeben und bei Seitenanfragen das Template `app/view/404.html`, folgen Sie dem nachstehenden Beispiel.

> Das Beispiel verwendet native PHP-Templates. Andere Templates wie `twig`, `blade`, `think-template` funktionieren nach demselben Prinzip.

**Erstellen Sie die Datei `app/view/404.html`**
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

**Fügen Sie folgenden Code in `config/route.php` ein:**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // Bei AJAX-Anfragen JSON zurückgeben
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // Bei Seitenanfragen das 404.html-Template zurückgeben
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# Benutzerdefinierte 405

Seit webman-framework 1.5.23 unterstützt der Fallback-Callback einen `status`-Parameter. 404 bedeutet, dass die Anfrage nicht existiert; 405 bedeutet, dass die aktuelle Anfragemethode nicht unterstützt wird (z.B. Zugriff per GET auf eine mit `Route::post()` definierte Route).

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

# Benutzerdefinierte 500

**Erstellen Sie `app/view/500.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
Benutzerdefiniertes Fehlertemplate:
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**Erstellen Sie `app/exception/Handler.php`** (Erstellen Sie das Verzeichnis, falls es nicht existiert)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * Antwort rendern und zurückgeben
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // Bei AJAX-Anfragen JSON-Daten zurückgeben
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // Bei Seitenanfragen das 500.html-Template zurückgeben
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**Konfigurieren Sie `config/exception.php`**
```php
return [
    '' => \app\exception\Handler::class,
];
```
