# Пользовательская 404

Если вы хотите динамически управлять содержимым 404, например возвращать JSON-данные `{"code:"404", "msg":"404 not found"}` при AJAX-запросах и возвращать шаблон `app/view/404.html` при запросах страниц, ознакомьтесь со следующим примером.

> Ниже приведён пример на нативных PHP-шаблонах. Другие шаблоны `twig`, `blade`, `think-template` работают по тому же принципу.

**Создать файл `app/view/404.html`**
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

**Добавить следующий код в `config/route.php`:**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // Возвращать JSON при AJAX-запросах
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // Возвращать шаблон 404.html при запросах страницы
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# Пользовательская 405

Начиная с webman-framework 1.5.23, callback fallback поддерживает параметр `status`. 404 означает, что запрос не существует; 405 — что текущий метод запроса не поддерживается (например, доступ по GET к маршруту, определённому через `Route::post()`).

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

# Пользовательская 500

**Создать `app/view/500.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
Пользовательский шаблон ошибки:
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**Создать `app/exception/Handler.php`** (создать каталог, если его нет)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * Рендеринг и возврат ответа
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // Возвращать JSON-данные при AJAX-запросах
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // Возвращать шаблон 500.html при запросах страницы
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**Настроить `config/exception.php`**
```php
return [
    '' => \app\exception\Handler::class,
];
```
