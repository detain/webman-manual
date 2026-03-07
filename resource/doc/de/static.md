# webman – Behandlung statischer Dateien

webman unterstützt den Zugriff auf statische Dateien, die alle im Verzeichnis `public` abgelegt sind. Beispielsweise bedeutet der Zugriff auf `http://127.0.0.8787/upload/avatar.png` tatsächlich den Zugriff auf `{Hauptprojektverzeichnis}/public/upload/avatar.png`.

> **Hinweis**
> Der Zugriff auf statische Dateien, der mit `/app/xx/Dateiname` beginnt, erfolgt tatsächlich auf das `public`-Verzeichnis des Anwendungs-Plugins. Das heißt, der Zugriff auf Verzeichnisse unter `{Hauptprojektverzeichnis}/public/app/` wird nicht unterstützt.
> Weitere Informationen finden Sie unter [Anwendungs-Plugins](./plugin/app.md).

## Deaktivieren der Unterstützung für statische Dateien

Wenn keine Unterstützung für statische Dateien benötigt wird, öffnen Sie die Datei `config/static.php` und ändern Sie die Option `enable` auf `false`. Nach der Deaktivierung wird bei allen Zugriffen auf statische Dateien ein 404-Fehler zurückgegeben.

## Ändern des Verzeichnisses für statische Dateien

Standardmäßig verwendet webman das Verzeichnis `public` für statische Dateien. Zum Ändern bearbeiten Sie die Hilfsfunktion `public_path()` in der Datei `support/helpers.php`.

## Middleware für statische Dateien

webman enthält eine Middleware für statische Dateien unter `app/middleware/StaticFile.php`.
Gelegentlich müssen statische Dateien behandelt werden, z. B. um Cross-Origin-HTTP-Header hinzuzufügen oder den Zugriff auf Dateien zu verbieten, die mit einem Punkt (`.`) beginnen. Hierfür kann diese Middleware verwendet werden.

Der Inhalt von `app/middleware/StaticFile.php` entspricht etwa Folgendem:
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
        // Zugriff auf versteckte Dateien, die mit einem Punkt beginnen, verbieten
        if (strpos($request->path(), '/.') !== false) {
            return response('<h1>403 verboten</h1>', 403);
        }
        /** @var Response $response */
        $response = $next($request);
        // Cross-Origin-Header hinzufügen
        /*$response->withHeaders([
            'Access-Control-Allow-Origin'      => '*',
            'Access-Control-Allow-Credentials' => 'true',
        ]);*/
        return $response;
    }
}
```
Wenn diese Middleware benötigt wird, muss sie in der Datei `config/static.php` unter der Option `middleware` aktiviert werden.
