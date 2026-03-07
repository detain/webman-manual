# MongoDB

webman по умолчанию использует [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) в качестве компонента MongoDB. Этот компонент извлечён из проекта Laravel и работает так же, как в Laravel.

Перед использованием `mongodb/laravel-mongodb` необходимо установить расширение MongoDB для `php-cli`.

> **Примечание**
> Используйте команду `php -m | grep mongodb` для проверки, установлено ли расширение MongoDB для `php-cli`. Обратите внимание: даже если вы установили расширение MongoDB для `php-fpm`, это не означает, что вы можете использовать его для `php-cli`, поскольку `php-cli` и `php-fpm` — это разные приложения, которые могут использовать разные конфигурационные файлы `php.ini`. Чтобы узнать, какой файл конфигурации `php.ini` используется для вашего `php-cli`, используйте команду `php --ini`.

## Установка

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

После установки необходимо выполнить restart (reload недостаточен).

## Настройка
Добавьте соединение `mongodb` в файле `config/database.php`, примерно так:

```php
return [

    'default' => 'mysql',

    'connections' => [

         ...здесь опущены другие настройки...

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => '127.0.0.1',
            'port'     =>  27017,
            'database' => 'test',
            'username' => null,
            'password' => null,
            'options' => [
                // здесь вы можете передать дополнительные настройки в Mongo Driver Manager
                // см. https://www.php.net/manual/en/mongodb-driver-manager.construct.php раздел "Uri Options" для списка всех параметров, которые вы можете использовать

                'appname' => 'homestead'
            ],
        ],
    ],
];
```

## Пример
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        Db::connection('mongodb')->table('test')->insert([1,2,3]);
        return json(Db::connection('mongodb')->table('test')->get());
    }
}
```

## Пример модели
```php
<?php
namespace app\model;

use DateTimeInterface;
use support\MongoModel as Model;

class Test extends Model
{
    protected $connection = 'mongodb';

    protected $table = 'test';

    public $timestamps = true;

    /**
     * @param DateTimeInterface $date
     * @return string
     */
    protected function serializeDate(DateTimeInterface $date): string
    {
        return $date->format('Y-m-d H:i:s');
    }
}
```

## Дополнительная информация

Для получения дополнительной информации посетите: https://github.com/mongodb/laravel-mongodb
