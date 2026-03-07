# Мультиязычность

Мультиязычная поддержка использует компонент [symfony/translation](https://github.com/symfony/translation).

## Установка
```
composer require symfony/translation
```

## Создание языковых пакетов
webman по умолчанию размещает языковые пакеты в каталоге `resource/translations` (создайте его, если не существует). Чтобы изменить каталог, настройте в `config/translation.php`.
Каждому языку соответствует подкаталог, определения языка по умолчанию хранятся в `messages.php`. Пример:
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

Все языковые файлы возвращают массив, например:
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## Конфигурация

`config/translation.php`

```php
return [
    // Язык по умолчанию
    'locale' => 'zh_CN',
    // Запасной язык: если перевод не найден в текущем языке, используется перевод из запасного языка
    'fallback_locale' => ['zh_CN', 'en'],
    // Каталог для хранения языковых файлов
    'path' => base_path() . '/resource/translations',
];
```

## Перевод

Для перевода используется метод `trans()`.

Создайте языковой файл `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 世界!',
];
```

Создайте файл `app/controller/UserController.php`:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        $hello = trans('hello'); // 你好 世界!
        return response($hello);
    }
}
```

При обращении к `http://127.0.0.1:8787/user/get` будет возвращено «你好 世界!»

## Изменение языка по умолчанию

Для переключения языка используйте метод `locale()`.

Добавьте языковой файл `resource/translations/en/messages.php`:
```php
return [
    'hello' => 'hello world!',
];
```

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // Переключить язык
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
При обращении к `http://127.0.0.1:8787/user/get` будет возвращено «hello world!»

Можно также использовать 4-й параметр функции `trans()` для временного переключения языка. Например, приведённый выше пример эквивалентен следующему:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // 4-й параметр переключает язык
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## Явная установка языка для каждого запроса
translation — синглтон, то есть все запросы используют один экземпляр. Если запрос устанавливает язык по умолчанию через `locale()`, это повлияет на все последующие запросы процесса. Поэтому следует явно устанавливать язык для каждого запроса, например с помощью следующего middleware:

Создайте файл `app/middleware/Lang.php` (создайте каталог, если не существует):
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Lang implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        locale(session('lang', 'zh_CN'));
        return $handler($request);
    }
}
```

Добавьте глобальный middleware в `config/middleware.php`:
```php
return [
    // Глобальный middleware
    '' => [
        // ... остальные middleware опущены
        app\middleware\Lang::class,
    ]
];
```


## Использование плейсхолдеров
Иногда сообщение содержит переменные, подлежащие переводу, например
```php
trans('hello ' . $name);
```
В таких случаях используются плейсхолдеры.

Измените `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 %name%!',
];
```
При переводе передавайте значения через второй параметр:
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## Обработка множественного числа
В некоторых языках структура предложения зависит от количества. Например, «There is %count% apple» верно при `%count%` = 1, но неверно при большем значении.

В таких случаях используется **вертикальная черта** (`|`) для перечисления форм множественного числа.

Добавьте `apple_count` в файл `resource/translations/en/messages.php`:
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

Можно также задать числовые диапазоны для более сложных правил множественного числа:
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## Указание языкового файла

Файл по умолчанию называется `messages.php`, но можно создавать файлы с другими именами.

Создайте файл `resource/translations/zh_CN/admin.php`:
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

Укажите файл третьим параметром `trans()` (расширение `.php` опускается):
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## Дополнительная информация
См. [документацию symfony/translation](https://symfony.com/doc/current/translation.html)
