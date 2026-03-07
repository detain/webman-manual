# webman – Обработка статических файлов

webman поддерживает доступ к статическим файлам, которые находятся в каталоге `public`. Например, при доступе по адресу `http://127.0.0.8787/upload/avatar.png` фактически осуществляется доступ к файлу `{основной_каталог_проекта}/public/upload/avatar.png`.

> **Обратите внимание**
> Доступ к статическим файлам, начинающийся с `/app/xx/имя_файла`, фактически осуществляется к каталогу `public` приложения-плагина. То есть доступ к каталогам в `{основной_каталог_проекта}/public/app/` не поддерживается.
> Подробнее см. в [Плагинах приложений](./plugin/app.md).

## Отключение поддержки статических файлов

Если не требуется поддержка статических файлов, откройте `config/static.php` и измените параметр `enable` на `false`. После отключения любой доступ к статическим файлам будет возвращать ошибку 404.

## Изменение каталога статических файлов

По умолчанию webman использует каталог `public` для статических файлов. Для изменения отредактируйте функцию-помощник `public_path()` в файле `support/helpers.php`.

## Middleware для статических файлов

webman поставляется с middleware для статических файлов, расположенным в `app/middleware/StaticFile.php`.
Иногда требуется обработка статических файлов, например добавление HTTP-заголовков для кросс-доменных запросов или запрет доступа к файлам, начинающимся с точки (`.`). Для этого можно использовать это middleware.

Содержимое файла `app/middleware/StaticFile.php` выглядит примерно так:
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
        // Запретить доступ к скрытым файлам, начинающимся с точки
        if (strpos($request->path(), '/.') !== false) {
            return response('<h1>403 forbidden</h1>', 403);
        }
        /** @var Response $response */
        $response = $next($request);
        // Добавить заголовки для кросс-доменных запросов
        /*$response->withHeaders([
            'Access-Control-Allow-Origin'      => '*',
            'Access-Control-Allow-Credentials' => 'true',
        ]);*/
        return $response;
    }
}
```
Если требуется использование этого middleware, включите его в параметре `middleware` файла `config/static.php`.
