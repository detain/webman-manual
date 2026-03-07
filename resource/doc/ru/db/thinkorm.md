# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) — компонент базы данных на базе [top-think/think-orm](https://github.com/top-think/think-orm). Поддерживает пул соединений и работает как в корутинной, так и в некорутинной среде.

## Установка

`composer require -W webman/think-orm`

После установки необходимо выполнить restart (перезапуск). reload не применяется.

## Файл конфигурации

Измените файл конфигурации `config/think-orm.php` в соответствии с вашими потребностями.

## Документация

https://www.kancloud.cn/manual/think-orm

## Использование

```php
<?php
namespace app\controller;

use support\Request;
use support\think\Db;

class FooController
{
    public function get(Request $request)
    {
        $user = Db::table('user')->where('uid', '>', 1)->find();
        return json($user);
    }
}
```

## Создание моделей

Модели think-orm наследуют `support\think\Model`, как показано ниже:

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * Таблица, связанная с моделью.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * Первичный ключ таблицы.
     *
     * @var string
     */
    protected $pk = 'id';

}
```

Вы также можете создавать модели think-orm следующей командой:

```
php webman make:model имя_таблицы
```

> **Подсказка**
> Для этой команды требуется `webman/console`. Установите его: `composer require webman/console ^1.2.13`.

> **Обратите внимание**
> Если `make:model` обнаруживает, что основной проект использует `illuminate/database`, будут созданы файлы моделей на базе Illuminate, а не think-orm. В этом случае добавьте параметр `tp`: `php webman make:model имя_таблицы tp` (если не сработает, обновите `webman/console`).
