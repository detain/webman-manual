# Контроллер

Создайте новый файл контроллера `app/controller/FooController.php`.

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

При доступе к `http://127.0.0.1:8787/foo` страница вернёт `hello index`.

При доступе к `http://127.0.0.1:8787/foo/hello` страница вернёт `hello webman`.

Конечно, вы можете изменить правила маршрутизации через конфигурацию маршрутов, см. [Маршруты](route.md).

> **Совет**
> Если возникает ошибка 404, откройте `config/app.php`, установите `controller_suffix` в `Controller` и перезапустите.

## Суффикс контроллера
Начиная с версии 1.3, webman поддерживает установку суффикса контроллера в `config/app.php`. Если `controller_suffix` в `config/app.php` установлен в пустую строку `''`, контроллер будет выглядеть так:

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

Настоятельно рекомендуется установить суффикс контроллера как `Controller`, чтобы избежать конфликта имён с моделями и повысить безопасность.

## Пояснение
- Фреймворк автоматически передаёт объект `support\Request` в контроллер, через который можно получить данные пользователя (get, post, header, cookie и т.д.), см. [Запросы](request.md).
- Контроллер может возвращать числа, строки или объекты `support\Response`, но не другие типы данных.
- Объекты `support\Response` создаются с помощью функций `response()`, `json()`, `xml()`, `jsonp()`, `redirect()` и т.д.

## Привязка параметров контроллера

#### Пример
webman поддерживает автоматическую привязку параметров запроса к параметрам методов контроллера. Например:

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

Значения `name` и `age` можно передать через `GET` или `POST`, либо через параметры маршрута. Например:

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

Приоритет: `параметры маршрута` > `GET` > `POST`.

#### Значения по умолчанию

При обращении к `/user/create?name=tom` появится ошибка:

```html
Missing input parameter age
```

Причина — не передан параметр `age`. Решение — задать значение по умолчанию. Например:

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

#### Типы параметров
При обращении к `/user/create?name=tom&age=not_int` появится ошибка:

> **Совет**
> Для удобства тестирования параметры указаны в адресной строке. В реальной разработке их следует передавать через `POST`.

```html
Input age must be of type int, string given
```

Полученные данные преобразуются по типу. При невозможности преобразования выбрасывается `support\exception\InputTypeException`. Поскольку `age` нельзя преобразовать в `int`, возникает эта ошибка.

#### Пользовательские сообщения об ошибках
Сообщения вроде `Missing input parameter age` и `Input age must be of type int, string given` можно настроить через переводы. См. команды:

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => 'Входной параметр :parameter должен быть типа :exceptType, передан тип :actualType',
    'Missing input parameter :parameter' => 'Отсутствует входной параметр :parameter',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### Другие типы
webman поддерживает типы `int`, `float`, `string`, `bool`, `array`, `object` и экземпляры классов. Например:

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

При обращении к `/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar` получите:

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

#### Экземпляр класса
webman поддерживает передачу экземпляров классов через подсказки типов. Например:

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

При обращении к `/blog/create?blog[title]=hello&blog[content]=world` получите:

```json
{
  "title": "hello",
  "content": "world"
}
```

#### Экземпляр модели

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
    // Определите здесь заполняемые поля, чтобы предотвратить передачу небезопасных полей с фронтенда
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

При обращении к `/user/create?user[name]=tom&user[age]=18` получите результат вроде:

```json
1
```

## Жизненный цикл контроллера

Когда `controller_reuse` в `config/app.php` равен `false`, каждый запрос инициализирует экземпляр контроллера один раз, после завершения запроса экземпляр уничтожается. Это аналогично традиционным фреймворкам.

Когда `controller_reuse` равен `true`, все запросы используют один и тот же экземпляр контроллера, который после создания остаётся в памяти.

> **Внимание**
> При повторном использовании контроллера запросы не должны менять его свойства, так как это повлияет на последующие запросы. Например:

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
        // Метод сохранит модель после первого запроса update?id=1
        // При запросе delete?id=2 будут удалены данные id=1
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **Совет**
> Возврат данных в конструкторе `__construct()` контроллера не даёт эффекта. Например:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // return в конструкторе не имеет эффекта, браузер не получит этот ответ
        return response('hello'); 
    }
}
```

## Различия между повторным использованием и без

#### Без повторного использования
Каждый запрос создаёт новый экземпляр контроллера, который освобождается после завершения. Привычный режим традиционных фреймворков. Производительность чуть ниже (около 10 % в тесте helloworld, в реальных задачах обычно несущественно).

#### С повторным использованием
Экземпляр создаётся один раз на процесс и сохраняется. Последующие запросы его переиспользуют. Лучшая производительность, но менее привычно для многих разработчиков.

#### Повторное использование невозможно, когда:

Запрос меняет свойства контроллера — изменения повлияют на следующие запросы.

Инициализация выполняется в `__construct()` для каждого запроса — конструктор вызывается один раз на процесс, а не при каждом запросе.
