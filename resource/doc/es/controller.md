# Controlador

Cree un nuevo archivo de controlador `app/controller/FooController.php`.

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

Al acceder a `http://127.0.0.1:8787/foo`, la página devuelve `hello index`.

Al acceder a `http://127.0.0.1:8787/foo/hello`, la página devuelve `hello webman`.

Puede cambiar las reglas de enrutamiento mediante la configuración de rutas, consulte [Rutas](route.md).

> **Consejo**
> Si aparece un error 404, abra `config/app.php`, establezca `controller_suffix` en `Controller` y reinicie.

## Sufijo del controlador
A partir de la versión 1.3, webman admite configurar el sufijo del controlador en `config/app.php`. Si `controller_suffix` está configurado como cadena vacía `''`, el controlador tendrá este aspecto:

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

Se recomienda encarecidamente establecer el sufijo del controlador en `Controller` para evitar conflictos con nombres de clases de modelos y mejorar la seguridad.

## Explicación
- El framework pasa automáticamente el objeto `support\Request` al controlador, con el que puede obtener datos de entrada del usuario (get, post, header, cookie, etc.), consulte [Solicitud](request.md).
- El controlador puede devolver números, cadenas u objetos `support\Response`, pero no otros tipos de datos.
- Los objetos `support\Response` se pueden crear con funciones auxiliares como `response()`, `json()`, `xml()`, `jsonp()`, `redirect()`, etc.

## Enlace de parámetros del controlador

#### Ejemplo
webman permite enlazar automáticamente los parámetros de la solicitud a los parámetros del método del controlador. Por ejemplo:

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

Puede pasar los valores de `name` y `age` mediante `GET` o `POST`, o a través de parámetros de ruta. Por ejemplo:

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

La prioridad es: `parámetros de ruta` > `GET` > `POST`.

#### Valores por defecto

Al acceder a `/user/create?name=tom`, obtendrá el siguiente error:

```html
Missing input parameter age
```

El motivo es que no se pasó el parámetro `age`. Puede solucionarlo definiendo un valor por defecto. Por ejemplo:

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

#### Tipos de parámetros
Al acceder a `/user/create?name=tom&age=not_int`, obtendrá el siguiente error:

> **Consejo**
> Para facilitar las pruebas, pasamos los parámetros directamente en la barra de direcciones. En desarrollo real, los parámetros deberían pasarse mediante `POST`.

```html
Input age must be of type int, string given
```

Los datos recibidos se convierten según el tipo. Si la conversión falla, se lanza la excepción `support\exception\InputTypeException`. Como `age` no puede convertirse a `int`, aparece este error.

#### Mensajes de error personalizados
Puede personalizar mensajes como `Missing input parameter age` e `Input age must be of type int, string given` mediante traducción. Consulte los siguientes comandos:

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => 'El parámetro de entrada :parameter debe ser de tipo :exceptType, el tipo recibido es :actualType',
    'Missing input parameter :parameter' => 'Falta el parámetro de entrada :parameter',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### Otros tipos
webman admite tipos como `int`, `float`, `string`, `bool`, `array`, `object` e `instancias de clase`. Por ejemplo:

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

Al acceder a `/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar`, obtendrá:

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

#### Instancia de clase
webman permite pasar instancias de clases mediante type hints. Por ejemplo:

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

Al acceder a `/blog/create?blog[title]=hello&blog[content]=world`, obtendrá:

```json
{
  "title": "hello",
  "content": "world"
}
```

#### Instancia de modelo

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
    // Definir aquí los campos rellenables para evitar campos inseguros del frontend
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

Al acceder a `/user/create?user[name]=tom&user[age]=18`, obtendrá un resultado similar a:

```json
1
```

## Ciclo de vida del controlador

Cuando `controller_reuse` en `config/app.php` es `false`, cada solicitud inicializa una vez la instancia del controlador, que se destruye al finalizar. Es el comportamiento típico de frameworks tradicionales.

Cuando `controller_reuse` en `config/app.php` es `true`, todas las solicitudes reutilizan la misma instancia del controlador, que permanece en memoria una vez creada.

> **Nota**
> Con la reutilización activada, las solicitudes no deben modificar propiedades del controlador, ya que afectaría a solicitudes posteriores. Por ejemplo:

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
        // Este método retiene el modelo tras la primera solicitud update?id=1
        // Si llega otra solicitud delete?id=2, se eliminarán los datos del id 1
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **Consejo**
> Devolver datos en el constructor `__construct()` del controlador no tiene efecto. Por ejemplo:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // El return en el constructor no tiene efecto; el navegador no recibirá esta respuesta
        return response('hello'); 
    }
}
```

## Diferencia entre no reutilizar y reutilizar el controlador

#### Sin reutilización
Cada solicitud crea una nueva instancia que se libera al finalizar. Comportamiento habitual en frameworks tradicionales. Rendimiento ligeramente inferior (aprox. 10 % en benchmark helloworld, normalmente insignificante en proyectos reales).

#### Con reutilización
Se crea una instancia por proceso y se mantiene en memoria. Las solicitudes siguientes la reutilizan. Mejor rendimiento, pero menos habitual para muchos desarrolladores.

#### No es posible reutilizar cuando:

La solicitud modifica propiedades del controlador; esos cambios afectarían a solicitudes posteriores.

Se realiza inicialización en `__construct()` para cada solicitud; el constructor solo se ejecuta una vez por proceso, no en cada solicitud.
