# Быстрый старт с базой данных (на основе компонента Laravel)

[webman/database](https://github.com/webman-php/database) построен на [illuminate/database](https://github.com/illuminate/database) и добавляет пул соединений для корутин и не-корутин сред. Использование такое же, как в Laravel.

Также можно обратиться к разделу [Использование других компонентов базы данных](others.md), чтобы использовать ThinkPHP или другие базы данных.

## Установка базы данных

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

После установки требуется перезапуск (restart), reload не сработает.

> **Подсказка**
> webman/database зависит от `illuminate/database` Laravel, поэтому при установке автоматически устанавливаются зависимости `illuminate/database`.

> **Примечание**
> Если не нужны пагинация, события БД и логирование SQL, достаточно выполнить:
> `composer require -W webman/database`

## Конфигурация базы данных
`config/database.php`
```php

return [
    // База данных по умолчанию
    'default' => 'mysql',

    // Настройки подключений
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'test',
            'username'    => 'root',
            'password'    => '',
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
            'options' => [
                PDO::ATTR_EMULATE_PREPARES => false, // Обязательно при использовании swoole или swow
            ],
            'pool' => [ // Пул соединений
                'max_connections' => 5, // Максимальное число соединений
                'min_connections' => 1, // Минимальное число соединений
                'wait_timeout' => 3,    // Максимальное время ожидания соединения из пула; при превышении — исключение. Только в корутинной среде
                'idle_timeout' => 60,   // Максимальное время простоя соединений; затем они закрываются до min_connections
                'heartbeat_interval' => 50, // Интервал проверки пула (сек). Рекомендуется меньше 60
            ],
        ],
    ],
];
```

Кроме настроек `pool`, остальное совпадает с Laravel.

## О пуле соединений
* У каждого процесса свой пул соединений, между процессами пул не делится.
* Без корутин запросы выполняются последовательно, нет конкурентности, поэтому в пуле не более одного соединения.
* С корутинами запросы выполняются параллельно; пул подстраивает число соединений под нагрузку, не более `max_connections` и не менее `min_connections`.
* Из-за лимита `max_connections`, когда корутин, работающих с БД, больше, часть ждёт в очереди до `wait_timeout` секунд; при превышении генерируется исключение.
* В режиме простоя (и с корутинами, и без) соединения возвращаются в пул после `idle_timeout`, пока не достигнут `min_connections` (`min_connections` может быть 0).


## Пример использования базы данных
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        $default_uid = 29;
        $uid = $request->get('uid', $default_uid);
        $name = Db::table('users')->where('uid', $uid)->value('username');
        return response("hello $name");
    }
}
```

Использование как в Laravel: метод `Db::table()` для работы с базой данных.
