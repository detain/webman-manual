# Файлы конфигурации

## Расположение
Файлы конфигурации webman находятся в директории `config/`. Для доступа к соответствующим настройкам в проекте можно использовать функцию `config()`.

## Доступ к конфигурации

Получить все настройки:
```php
config();
```

Получить все настройки из `config/app.php`:
```php
config('app');
```

Получить настройку `debug` из `config/app.php`:
```php
config('app.debug');
```

Если конфигурация является массивом, можно использовать `.` для доступа к вложенным значениям. Например:
```php
config('file.key1.key2');
```

## Значение по умолчанию
```php
config($key, $default);
```
Передайте значение по умолчанию вторым параметром. Если конфигурация не существует, будет возвращено значение по умолчанию. Если конфигурация не существует и значение по умолчанию не задано, возвращается `null`.

## Пользовательская конфигурация
Разработчики могут добавлять собственные файлы конфигурации в директорию `config/`. Например:

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**Использование при доступе к конфигурации**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## Изменение конфигурации
Webman не поддерживает динамическое изменение конфигурации. Все настройки должны изменяться вручную в соответствующих файлах конфигурации, после чего нужно перезагрузить или перезапустить приложение.

> **Примечание**
> Конфигурация сервера `config/server.php` и конфигурация процессов `config/process.php` не поддерживают перезагрузку. Для применения изменений необходимо перезапустить приложение.

## Важное напоминание
При создании файлов конфигурации в поддиректории `config/`, например `config/order/status.php`, в директории `config/order/` должен быть файл `app.php` со следующим содержимым:
```php
<?php
return [
    'enable' => true,
];
```
Установка `enable` в `true` указывает фреймворку загружать конфигурации из этой директории.
Структура директории конфигурации должна выглядеть так:
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
Далее можно обращаться к массиву или отдельным ключам из `status.php` через `config.order.status`.


## Справочник по файлам конфигурации

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // Порт прослушивания (удалено с 1.6.0, настраивается в config/process.php)
    'transport' => 'tcp', // Протокол транспорта (удалено с 1.6.0, настраивается в config/process.php)
    'context' => [], // SSL и т.д. (удалено с 1.6.0, настраивается в config/process.php)
    'name' => 'webman', // Имя процесса (удалено с 1.6.0, настраивается в config/process.php)
    'count' => cpu_count() * 4, // Количество процессов (удалено с 1.6.0, настраивается в config/process.php)
    'user' => '', // Пользователь (удалено с 1.6.0, настраивается в config/process.php)
    'group' => '', // Группа (удалено с 1.6.0, настраивается в config/process.php)
    'reusePort' => false, // Включить повторное использование порта (удалено с 1.6.0, настраивается в config/process.php)
    'event_loop' => '',  // Класс event loop, по умолчанию выбирается автоматически
    'stop_timeout' => 2, // Максимальное время ожидания при получении сигнала stop/restart/reload, принудительный выход, если процесс не завершится вовремя
    'pid_file' => runtime_path() . '/webman.pid', // Расположение файла PID
    'status_file' => runtime_path() . '/webman.status', // Расположение файла состояния
    'stdout_file' => runtime_path() . '/logs/stdout.log', // Расположение файла stdout, весь вывод после запуска webman записывается сюда
    'log_file' => runtime_path() . '/logs/workerman.log', // Расположение лог-файла Workerman
    'max_package_size' => 10 * 1024 * 1024 // Максимальный размер пакета, 10M. Размер загружаемых файлов ограничен этим значением
];
```

#### app.php
```php
return [
    'debug' => true,  // Режим отладки, включает stack trace и т.д. при ошибках. Должен быть отключен в продакшене
    'error_reporting' => E_ALL, // Уровень сообщений об ошибках
    'default_timezone' => 'Asia/Shanghai', // Часовой пояс по умолчанию
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // Путь к публичной директории
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // Путь к директории runtime
    'controller_suffix' => 'Controller', // Суффикс контроллера
    'controller_reuse' => false, // Переиспользовать ли контроллеры
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // Конфигурация процессов webman
    'webman' => [ 
        'handler' => Http::class, // Класс обработчика процесса
        'listen' => 'http://0.0.0.0:8787', // Адрес прослушивания
        'count' => cpu_count() * 4, // Количество процессов, по умолчанию 4x CPU
        'user' => '', // Пользователь процесса, следует использовать пользователя с минимальными правами
        'group' => '', // Группа процесса, следует использовать группу с минимальными правами
        'reusePort' => false, // Включить reusePort, распределяет подключения между worker-процессами
        'eventLoop' => '', // Класс event loop, при пустом значении используется server.event_loop
        'context' => [], // Контекст прослушивания, напр. SSL
        'constructor' => [ // Параметры конструктора для обработчика процесса, здесь класс Http
            'requestClass' => Request::class, // Пользовательский класс запроса
            'logger' => Log::channel('default'), // Экземпляр логгера
            'appPath' => app_path(), // Путь к директории приложения
            'publicPath' => public_path() // Путь к публичной директории
        ]
    ],
    // Процесс мониторинга для авто-перезагрузки при изменении файлов и обнаружения утечек памяти
    'monitor' => [
        'handler' => app\process\Monitor::class, // Класс обработчика
        'reloadable' => false, // Этот процесс не выполняет reload
        'constructor' => [ // Параметры конструктора для обработчика процесса
            // Директории для наблюдения, избегайте слишком многих — это замедляет обнаружение
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Расширения файлов для отслеживания изменений
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // Прочие опции
            'options' => [
                // Включить мониторинг файлов, только Linux, по умолчанию отключен в daemon-режиме
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // Включить мониторинг памяти, только Linux
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// Возвращает экземпляр контейнера внедрения зависимостей PSR-11
return new Webman\Container;
```

#### dependence.php
```php
// Настроить сервисы и зависимости в контейнере внедрения зависимостей
return [];
```

#### route.php
```php

use support\Route;
// Определить маршрут для пути /test
Route::any('/test', function (Request $request) {
    return response('test');
});
```

#### view.php
```php
use support\view\Raw;
use support\view\Twig;
use support\view\Blade;
use support\view\ThinkPHP;

return [
    'handler' => Raw::class // Класс обработчика представлений по умолчанию
];
```

### autoload.php
```php
// Настроить файлы автозагрузки фреймворка
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php',
        base_path() . '/support/Response.php',
    ]
];
```

#### cache.php
```php
// Конфигурация кэша
return [
    'default' => 'file', // Драйвер по умолчанию
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // Путь хранения файлов кэша
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // Имя подключения Redis, ссылается на конфиг в redis.php
        ],
        'array' => [
            'driver' => 'array' // Кэш в памяти, очищается при перезапуске
        ]
    ]
];
```

#### redis.php
```php
return [
    'default' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 0,
    ],
];
```

#### database.php
```php
return [
 // База данных по умолчанию
 'default' => 'mysql',
 // Конфигурации подключений к базе данных
 'connections' => [

     'mysql' => [
         'driver'      => 'mysql',
         'host'        => '127.0.0.1',
         'port'        => 3306,
         'database'    => 'webman',
         'username'    => 'webman',
         'password'    => '',
         'unix_socket' => '',
         'charset'     => 'utf8',
         'collation'   => 'utf8_unicode_ci',
         'prefix'      => '',
         'strict'      => true,
         'engine'      => null,
     ],

     'sqlite' => [
         'driver'   => 'sqlite',
         'database' => '',
         'prefix'   => '',
     ],

     'pgsql' => [
         'driver'   => 'pgsql',
         'host'     => '127.0.0.1',
         'port'     => 5432,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
         'schema'   => 'public',
         'sslmode'  => 'prefer',
     ],

     'sqlsrv' => [
         'driver'   => 'sqlsrv',
         'host'     => 'localhost',
         'port'     => 1433,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
     ],
 ],
];
```

#### exception.php
```php
return [
    // Задать класс обработчика исключений
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // Обработчик
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // Имя лог-файла
                    7, //$maxFiles // Хранить логи 7 дней
                    Monolog\Logger::DEBUG, // Уровень логирования
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // Форматтер
                    'constructor' => [null, 'Y-m-d H:i:s', true], // Параметры форматтера
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // Тип
    'type' => 'file', // или redis или redis_cluster
     // Обработчик
    'handler' => FileSessionHandler::class,
     // Конфигурация
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // Директория хранения
        ],
        'redis' => [
            'host' => '127.0.0.1',
            'port' => 6379,
            'auth' => '',
            'timeout' => 2,
            'database' => '',
            'prefix' => 'redis_session_',
        ],
        'redis_cluster' => [
            'host' => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth' => '',
            'prefix' => 'redis_session_',
        ]
    ],
    'session_name' => 'PHPSID', // Имя сессии
    'auto_update_timestamp' => false, // Автообновление timestamp для предотвращения истечения сессии
    'lifetime' => 7*24*60*60, // Время жизни
    'cookie_lifetime' => 365*24*60*60, // Время жизни cookie
    'cookie_path' => '/', // Путь cookie
    'domain' => '', // Домен cookie
    'http_only' => true, // Только HTTP
    'secure' => false, // Только HTTPS
    'same_site' => '', // Атрибут SameSite
    'gc_probability' => [1, 1000], // Вероятность сборки мусора сессии
];
```

#### middleware.php
```php
// Настроить middleware
return [];
```

#### static.php
```php
return [
    'enable' => true, // Включить обслуживание статических файлов webman
    'middleware' => [ // Middleware статических файлов, для политики кэширования, CORS и т.д.
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // Язык по умолчанию
    'locale' => 'zh_CN',
    // Резервные языки
    'fallback_locale' => ['zh_CN', 'en'],
    // Расположение файлов переводов
    'path' => base_path() . '/resource/translations',
];
```
