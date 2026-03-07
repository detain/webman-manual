# База данных Medoo

[webman/medoo](https://github.com/webman-php/medoo) расширяет [Medoo](https://medoo.in/) поддержкой пула соединений и работает как в корутинной, так и в некорутинной среде. Использование аналогично Medoo.

## Установка
`composer require webman/medoo`

## Конфигурация базы данных Medoo
Расположение файла конфигурации: `config/plugin/webman/medoo/database.php`

## Использование базы данных Medoo
```php
<?php
namespace app\controller;

use support\Request;
use support\Medoo;

class Index
{
    public function index(Request $request)
    {
        $user = Medoo::get('user', '*', ['uid' => 1]);
        return json($user);
    }
}
```

> **Подсказка**
> `Medoo::get('user', '*', ['uid' => 1]);`
> эквивалентно
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## Конфигурация нескольких баз данных Medoo

**Конфигурация**
Добавьте новую конфигурацию в `config/plugin/webman/medoo/database.php` с любым ключом; здесь используется `other`.

```php
<?php
return [
    'default' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [ // Конфигурация пула соединений
            'max_connections' => 5, // Максимальное число соединений
            'min_connections' => 1, // Минимальное число соединений
            'wait_timeout' => 60,   // Максимальное время ожидания получения соединения из пула; исключение при превышении
            'idle_timeout' => 3,    // Максимальное время простоя соединений в пуле; при превышении закрываются до min_connections
            'heartbeat_interval' => 50, // Интервал heartbeat пула в секундах; рекомендуется менее 60 секунд
        ]
    ],
    // Добавление конфигурации other здесь
    'other' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [
            'max_connections' => 5,
            'min_connections' => 1,
            'wait_timeout' => 60,
            'idle_timeout' => 3,
            'heartbeat_interval' => 50,
        ],
    ],
];
```

## Использование базы данных Medoo
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

См. [официальную документацию Medoo](https://medoo.in/api/select)
