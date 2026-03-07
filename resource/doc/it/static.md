# webman – Gestione dei file statici

webman supporta l'accesso ai file statici, che sono tutti posizionati nella directory `public`. Ad esempio, accedere a `http://127.0.0.8787/upload/avatar.png` corrisponde effettivamente ad accedere a `{directory principale del progetto}/public/upload/avatar.png`.

> **Nota**
> L'accesso ai file statici che inizia con `/app/xx/nomefile` corrisponde in realtà all'accesso alla directory `public` del plugin dell'applicazione. In altre parole, non è supportato l'accesso alle directory sotto `{directory principale del progetto}/public/app/`.
> Per ulteriori informazioni, consultare il [plugin dell'applicazione](./plugin/app.md).

## Disabilitare il supporto dei file statici

Se non è necessario il supporto dei file statici, aprire il file `config/static.php` e impostare l'opzione `enable` su false. Dopo la disabilitazione, l'accesso a tutti i file statici restituirà un errore 404.

## Modificare la directory dei file statici

webman utilizza per impostazione predefinita la directory `public` come directory dei file statici. Per modificarla, modificare la funzione di supporto `public_path()` nel file `support/helpers.php`.

## Middleware per i file statici

webman include un middleware per i file statici, posizionato in `app/middleware/StaticFile.php`.
A volte è necessario eseguire alcune operazioni sui file statici, ad esempio aggiungere intestazioni HTTP per l'accesso cross-origin o impedire l'accesso ai file che iniziano con un punto (`.`). È possibile utilizzare questo middleware.

Il contenuto di `app/middleware/StaticFile.php` è simile al seguente:
```php
<?php
namespace support\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class StaticFile implements MiddlewareInterface
{
    public function process(Request $request, callable $next): Response
    {
        // Impedire l'accesso ai file nascosti che iniziano con .
        if (strpos($request->path(), '/.') !== false) {
            return response('<h1>403 forbidden</h1>', 403);
        }
        /** @var Response $response */
        $response = $next($request);
        // Aggiungere intestazioni HTTP per l'accesso cross-origin
        /*$response->withHeaders([
            'Access-Control-Allow-Origin'      => '*',
            'Access-Control-Allow-Credentials' => 'true',
        ]);*/
        return $response;
    }
}
```
Se è necessario utilizzare questo middleware, abilitarlo nell'opzione `middleware` del file `config/static.php`.
