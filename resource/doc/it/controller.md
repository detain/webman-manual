# Controller

Creare un nuovo file controller `app/controller/FooController.php`.

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

Quando si accede a `http://127.0.0.1:8787/foo`, la pagina restituisce `hello index`.

Quando si accede a `http://127.0.0.1:8787/foo/hello`, la pagina restituisce `hello webman`.

È possibile modificare le regole di routing tramite la configurazione delle route, vedere [Routing](route.md).

> **Suggerimento**
> In caso di errore 404, aprire `config/app.php`, impostare `controller_suffix` su `Controller` e riavviare.

## Suffisso del controller
Dalla versione 1.3, webman supporta l'impostazione del suffisso del controller in `config/app.php`. Se `controller_suffix` è impostato su stringa vuota `''`, il controller avrà questo aspetto:

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

Si consiglia vivamente di impostare il suffisso del controller su `Controller` per evitare conflitti con i nomi delle classi dei modelli e migliorare la sicurezza.

## Spiegazione
- Il framework passa automaticamente l'oggetto `support\Request` al controller, con il quale è possibile ottenere i dati in input (get, post, header, cookie, ecc.), vedere [Richiesta](request.md).
- Il controller può restituire numeri, stringhe o oggetti `support\Response`, ma non altri tipi di dati.
- Gli oggetti `support\Response` possono essere creati con funzioni helper come `response()`, `json()`, `xml()`, `jsonp()`, `redirect()`, ecc.

## Binding dei parametri del controller

#### Esempio
webman supporta il binding automatico dei parametri della richiesta ai parametri dei metodi del controller. Ad esempio:

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

È possibile passare i valori di `name` e `age` tramite `GET` o `POST`, oppure tramite i parametri della route. Ad esempio:

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

La priorità è: `parametri della route` > `GET` > `POST`.

#### Valori predefiniti

Accedendo a `/user/create?name=tom`, si otterrà il seguente errore:

```html
Missing input parameter age
```

Il motivo è che non è stato passato il parametro `age`. Si può risolvere impostando un valore predefinito. Ad esempio:

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

#### Tipi di parametri
Accedendo a `/user/create?name=tom&age=not_int`, si otterrà il seguente errore:

> **Suggerimento**
> Per comodità di test, passiamo i parametri direttamente nella barra degli indirizzi. In sviluppo reale i parametri dovrebbero essere passati tramite `POST`.

```html
Input age must be of type int, string given
```

I dati ricevuti vengono convertiti in base al tipo. Se la conversione fallisce, viene lanciata l'eccezione `support\exception\InputTypeException`. Poiché `age` non può essere convertito in `int`, appare questo errore.

#### Messaggi di errore personalizzati
È possibile personalizzare messaggi come `Missing input parameter age` e `Input age must be of type int, string given` tramite la traduzione. Fare riferimento ai seguenti comandi:

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => 'Il parametro di input :parameter deve essere di tipo :exceptType, il tipo passato è :actualType',
    'Missing input parameter :parameter' => 'Manca il parametro di input :parameter',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### Altri tipi
webman supporta tipi come `int`, `float`, `string`, `bool`, `array`, `object` e `istanze di classe`. Ad esempio:

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

Accedendo a `/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar`, si otterrà:

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

#### Istanza di classe
webman supporta il passaggio di istanze di classi tramite type hint. Ad esempio:

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

Accedendo a `/blog/create?blog[title]=hello&blog[content]=world`, si otterrà:

```json
{
  "title": "hello",
  "content": "world"
}
```

#### Istanza di modello

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
    // Definire qui i campi fillable per evitare campi non sicuri dal frontend
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

Accedendo a `/user/create?user[name]=tom&user[age]=18`, si otterrà un risultato simile a:

```json
1
```

## Ciclo di vita del controller

Quando `controller_reuse` in `config/app.php` è `false`, ogni richiesta inizializza una volta l'istanza del controller, che viene distrutta al termine. Comportamento analogo ai framework tradizionali.

Quando `controller_reuse` in `config/app.php` è `true`, tutte le richieste riutilizzano la stessa istanza del controller, che permane in memoria una volta creata.

> **Nota**
> Con il riutilizzo attivo, le richieste non devono modificare le proprietà del controller, poiché le modifiche influirebbero sulle richieste successive. Ad esempio:

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
        // Questo metodo conserva il modello dopo la prima richiesta update?id=1
        // Una richiesta delete?id=2 eliminerebbe i dati dell'id 1
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **Suggerimento**
> Restituire dati nel costruttore `__construct()` del controller non ha effetto. Ad esempio:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // Il return nel costruttore non ha effetto; il browser non riceverà questa risposta
        return response('hello'); 
    }
}
```

## Differenza tra controller senza e con riutilizzo

#### Senza riutilizzo
Ogni richiesta crea una nuova istanza del controller, liberata al termine. Comportamento tradizionale, familiare alla maggior parte degli sviluppatori. Prestazioni leggermente inferiori (circa 10% in benchmark helloworld, spesso trascurabile in produzione).

#### Con riutilizzo
Un'istanza viene creata una volta per processo e mantenuta. Le richieste successive la riutilizzano. Prestazioni migliori, ma meno usuale per molti sviluppatori.

#### Il riutilizzo non è possibile quando:

La richiesta modifica le proprietà del controller; le modifiche influirebbero sulle richieste successive.

Viene eseguita l'inizializzazione in `__construct()` per ogni richiesta; il costruttore viene chiamato una sola volta per processo, non per ogni richiesta.
