# Personalizar 404

Si desea controlar dinámicamente el contenido de 404, por ejemplo devolver datos JSON `{"code:"404", "msg":"404 not found"}` en solicitudes AJAX y devolver la plantilla `app/view/404.html` en solicitudes de página, consulte el siguiente ejemplo.

> El ejemplo utiliza plantillas nativas de PHP. Otras plantillas como `twig`, `blade`, `think-template` siguen el mismo principio.

**Crear el archivo `app/view/404.html`**
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

**Añadir el siguiente código en `config/route.php`:**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // Devolver JSON en solicitudes AJAX
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // Devolver la plantilla 404.html en solicitudes de página
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# Personalizar 405

Desde webman-framework 1.5.23, el callback de fallback admite el parámetro `status`. 404 indica que la solicitud no existe; 405 indica que el método de solicitud actual no está permitido (p. ej. acceder con GET a una ruta definida con `Route::post()`).

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

**Crear `app/view/500.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
Plantilla de error personalizada:
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**Crear `app/exception/Handler.php`** (crear el directorio si no existe)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * Renderizar y devolver la respuesta
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // Devolver datos JSON en solicitudes AJAX
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // Devolver la plantilla 500.html en solicitudes de página
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
