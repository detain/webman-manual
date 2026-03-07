# webman: Manejo de archivos estáticos

webman admite el acceso a archivos estáticos, que se colocan en el directorio `public`. Por ejemplo, al acceder a `http://127.0.0.8787/upload/avatar.png` en realidad se accede a `{directorio principal del proyecto}/public/upload/avatar.png`.

> **Nota**
> El acceso a archivos estáticos que comienza con `/app/xx/nombre_de_archivo` en realidad accede al directorio `public` del complemento de la aplicación. Es decir, no se admite el acceso a directorios bajo `{directorio principal del proyecto}/public/app/`.
> Para más información, consulte [Complementos de la aplicación](./plugin/app.md).

## Deshabilitar el soporte de archivos estáticos

Si no se necesita el soporte de archivos estáticos, abra `config/static.php` y cambie la opción `enable` a `false`. Una vez deshabilitado, cualquier acceso a archivos estáticos devolverá un error 404.

## Cambiar el directorio de archivos estáticos

Por defecto, webman utiliza el directorio `public` como directorio de archivos estáticos. Si necesita cambiarlo, modifique la función de ayuda `public_path()` en `support/helpers.php`.

## Middleware de archivos estáticos

webman incluye un middleware para archivos estáticos, ubicado en `app/middleware/StaticFile.php`.
A veces es necesario realizar cierto procesamiento en los archivos estáticos, como agregar encabezados HTTP para permitir el acceso desde otros dominios o prohibir el acceso a archivos que comienzan con punto (`.`). Puede utilizarse este middleware para ello.

El contenido de `app/middleware/StaticFile.php` es similar al siguiente:
```php
<?php
namespace support\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class StaticFile implements MiddlewareInterface
{
    public function process(Request $request, callable $next) : Response
    {
        // Prohibir el acceso a archivos ocultos que comienzan con .
        if (strpos($request->path(), '/.') !== false) {
            return response('<h1>403 prohibido</h1>', 403);
        }
        /** @var Response $response */
        $response = $next($request);
        // Agregar encabezados para acceso desde otros dominios
        /*$response->withHeaders([
            'Access-Control-Allow-Origin'      => '*',
            'Access-Control-Allow-Credentials' => 'true',
        ]);*/
        return $response;
    }
}
```
Si necesita este middleware, debe habilitarlo en la opción `middleware` en `config/static.php`.
