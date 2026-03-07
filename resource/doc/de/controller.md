# Controller

Erstellen Sie die neue Controller-Datei `app/controller/FooController.php`.

```php
<?php
namespace app\controller;

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

Bei Aufruf von `http://127.0.0.1:8787/foo` gibt die Seite `hello index` zurück.

Bei Aufruf von `http://127.0.0.1:8787/foo/hello` gibt die Seite `hello webman` zurück.

Natürlich können Sie die Routing-Regeln über die Routenkonfiguration ändern, siehe [Routen](route.md).

> **Hinweis**
> Bei einem 404-Fehler öffnen Sie `config/app.php`, setzen Sie `controller_suffix` auf `Controller` und starten Sie neu.

## Controller-Suffix
Ab Version 1.3 unterstützt webman die Einstellung eines Controller-Suffix in `config/app.php`. Wenn `controller_suffix` in `config/app.php` auf `''` gesetzt ist, sieht der Controller wie folgt aus:

`app\controller\Foo.php`.

```php
<?php
namespace app\controller;

use support\Request;

class Foo
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

Es wird dringend empfohlen, das Controller-Suffix auf `Controller` zu setzen, um Namenskonflikte mit Modellklassen zu vermeiden und die Sicherheit zu erhöhen.

## Erklärung
- Das Framework übergibt automatisch ein `support\Request`-Objekt an den Controller, über das Benutzereingaben (get, post, header, cookie usw.) abgerufen werden können, siehe [Anfrage](request.md).
- Der Controller kann Zahlen, Zeichenketten oder `support\Response`-Objekte zurückgeben, aber keine anderen Datentypen.
- `support\Response`-Objekte können mit Hilfsfunktionen wie `response()`, `json()`, `xml()`, `jsonp()`, `redirect()` usw. erstellt werden.

## Controller-Parameter-Bindung

#### Beispiel
webman unterstützt die automatische Bindung von Anfrageparametern an Controller-Methodenparameter. Zum Beispiel:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

Sie können die Werte für `name` und `age` per `GET` oder `POST` übergeben oder über Routenparameter. Zum Beispiel:

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

Die Priorität ist: `Routenparameter` > `GET` > `POST`-Parameter.

#### Standardwerte

Beim Aufruf von `/user/create?name=tom` erhalten Sie folgenden Fehler:

```html
Missing input parameter age
```

Der Grund ist, dass der Parameter `age` nicht übergeben wurde. Das kann durch einen Standardwert gelöst werden. Zum Beispiel:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age = 18): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

#### Parametertypen
Beim Aufruf von `/user/create?name=tom&age=not_int` erhalten Sie folgenden Fehler:

> **Hinweis**
> Zur einfachen Tests geben wir die Parameter direkt in der Adresszeile ein. In der Praxis sollten Parameter per `POST` übergeben werden.

```html
Input age must be of type int, string given
```

Die empfangenen Daten werden gemäß dem Typ konvertiert. Bei Konvertierungsfehlern wird eine `support\exception\InputTypeException` ausgelöst. Da `age` nicht in `int` konvertiert werden kann, erscheint dieser Fehler.

#### Fehlermeldungen anpassen
Fehlermeldungen wie `Missing input parameter age` und `Input age must be of type int, string given` können per Mehrsprachigkeit angepasst werden. Siehe folgende Befehle:

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => 'Eingabeparameter :parameter muss vom Typ :exceptType sein, übergebener Typ ist :actualType',
    'Missing input parameter :parameter' => 'Eingabeparameter :parameter fehlt',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### Weitere Typen
webman unterstützt Parametertypen wie `int`, `float`, `string`, `bool`, `array`, `object` und `Klasseninstanzen`. Zum Beispiel:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age, float $balance, bool $vip, array $extension): Response
    {
        return json([
            'name' => $name,
            'age' => $age,
            'balance' => $balance,
            'vip' => $vip,
            'extension' => $extension,
        ]);
    }
}
```

Beim Aufruf von `/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar` erhalten Sie:

```json
{
  "name": "tom",
  "age": 18,
  "balance": 100.5,
  "vip": true,
  "extension": {
    "foo": "bar"
  }
}
```

#### Klasseninstanz
webman unterstützt die Übergabe von Klasseninstanzen über Parametertyp-Hinweise. Zum Beispiel:

**app\service\Blog.php**
```php
<?php
namespace app\service;
class Blog
{
    private $title;
    private $content;
    public function __construct(string $title, string $content)
    {
        $this->title = $title;
        $this->content = $content;
    }
    public function get()
    {
        return [
            'title' => $this->title,
            'content' => $this->content,
        ];
    }
}
```

**app\controller\BlogController.php**
```php
<?php
namespace app\controller;
use app\service\Blog;
use support\Response;

class BlogController
{
    public function create(Blog $blog): Response
    {
        return json($blog->get());
    }
}
```

Beim Aufruf von `/blog/create?blog[title]=hello&blog[content]=world` erhalten Sie:

```json
{
  "title": "hello",
  "content": "world"
}
```

#### Modellinstanz

**app\model\User.php**
```php
<?php
namespace app\model;
use support\Model;
class User extends Model
{
    protected $connection = 'mysql';
    protected $table = 'user';
    protected $primaryKey = 'id';
    public $timestamps = false;
    // Hier müssen füllbare Felder definiert werden, um unsichere Felder vom Frontend zu verhindern
    protected $fillable = ['name', 'age'];
}
```

**app\controller\UserController.php**
```php
<?php
namespace app\controller;
use app\model\User;
class UserController
{
    public function create(User $user): int
    {
        $user->save();
        return $user->id;
    }
}
```

Beim Aufruf von `/user/create?user[name]=tom&user[age]=18` erhalten Sie etwa:

```json
1
```

## Controller-Lebenszyklus

Wenn `controller_reuse` in `config/app.php` `false` ist, wird pro Anfrage einmal eine Controller-Instanz erstellt und nach der Anfrage wieder freigegeben – wie bei traditionellen Frameworks.

Wenn `controller_reuse` in `config/app.php` `true` ist, wird die Controller-Instanz wiederverwendet: Sie bleibt im Speicher und wird für alle Anfragen genutzt.

> **Hinweis**
> Bei Controller-Wiederverwendung dürfen Anfragen keine Controller-Eigenschaften ändern, da dies folgende Anfragen beeinflusst. Zum Beispiel:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    protected $model;
    
    public function update(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->update();
        return response('ok');
    }
    
    public function delete(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->delete();
        return response('ok');
    }
    
    protected function getModel($id)
    {
        // Nach der ersten Anfrage update?id=1 bleibt das Modell gespeichert
        // Bei einer weiteren Anfrage delete?id=2 werden die Daten von 1 gelöscht
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **Hinweis**
> Ein `return` im Controller-Konstruktor `__construct()` hat keine Wirkung. Zum Beispiel:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // return im Konstruktor hat keine Wirkung, der Browser erhält diese Antwort nicht
        return response('hello'); 
    }
}
```

## Unterschied zwischen Controller ohne und mit Wiederverwendung

#### Ohne Wiederverwendung
Für jede Anfrage wird eine neue Controller-Instanz erstellt und nach der Anfrage freigegeben. Entspricht traditionellen Frameworks und den Gewohnheiten vieler Entwickler. Etwas schlechtere Leistung als mit Wiederverwendung (ca. 10 % bei Helloworld-Benchmarks, bei realem Einsatz meist vernachlässigbar).

#### Mit Wiederverwendung
Pro Prozess wird nur einmal eine Controller-Instanz erstellt und nicht freigegeben. Folgende Anfragen in diesem Prozess nutzen dieselbe Instanz. Bessere Leistung, aber ungewöhnlicher für viele Entwickler.

#### Wiederverwendung nicht möglich, wenn:

Eine Anfrage die Eigenschaften des Controllers ändert – diese Änderungen würden folgende Anfragen beeinflussen.

Einige Entwickler führen pro Anfrage im Konstruktor `__construct()` Initialisierungen durch. In diesem Fall darf keine Wiederverwendung genutzt werden, da der Konstruktor pro Prozess nur einmal aufgerufen wird, nicht bei jeder Anfrage.
