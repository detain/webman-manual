# Controller

Gemäß dem PSR4-Standard beginnt der Namespace für Controller-Klassen mit `plugin\{Plugin-Kennung}`, zum Beispiel

Erstellen Sie die Controller-Datei `plugin/foo/app/controller/FooController.php`.

```php
<?php
namespace plugin\foo\app\controller;

use support\Request;

class FooController
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

Beim Aufrufen von `http://127.0.0.1:8787/app/foo/foo` wird die Seite `hello index` zurückgegeben.

Beim Aufrufen von `http://127.0.0.1:8787/app/foo/foo/hello` wird die Seite `hello webman` zurückgegeben.


## URL-Zugriff
Die URL-Adresspfade der Anwendungs-Plugins beginnen alle mit `/app`, gefolgt von der Plugin-Kennung und dann dem jeweiligen Controller und der Methode.
Beispiel: Die URL-Adresse von `plugin\foo\app\controller\UserController` ist `http://127.0.0.1:8787/app/foo/user`.
