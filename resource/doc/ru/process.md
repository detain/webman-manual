# Пользовательские процессы

В webman вы можете создавать пользовательские прослушиватели или процессы, как в workerman.

> **Примечание**
> Пользователям Windows необходимо запускать webman командой `php windows.php` для работы пользовательских процессов.

## Пользовательский HTTP-сервер
Иногда может потребоваться изменить основной код HTTP-сервиса webman. В таких случаях можно использовать пользовательский процесс.

Например, создайте файл `app\Server.php`.

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // Здесь переопределите методы из Webman\App
}
```

Добавьте следующую конфигурацию в `config/process.php`.

```php
use Workerman\Worker;

return [
    // ... остальные настройки опущены ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Количество процессов
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Класс запроса
            'logger' => \support\Log::channel('default'), // Экземпляр логгера
            'appPath' => app_path(), // Расположение директории app
            'publicPath' => public_path() // Расположение директории public
        ]
    ]
];
```

> **Подсказка**
> Чтобы отключить встроенный HTTP-процесс webman, установите `listen=>''` в `config/server.php`.

## Пример пользовательского WebSocket-прослушивателя

Создайте `app/Pusher.php`.

```php
<?php
namespace app;

use Workerman\Connection\TcpConnection;

class Pusher
{
    public function onConnect(TcpConnection $connection)
    {
        echo "onConnect\n";
    }

    public function onWebSocketConnect(TcpConnection $connection, $http_buffer)
    {
        echo "onWebSocketConnect\n";
    }

    public function onMessage(TcpConnection $connection, $data)
    {
        $connection->send($data);
    }

    public function onClose(TcpConnection $connection)
    {
        echo "onClose\n";
    }
}
```

> Примечание: все методы onXXX должны быть public.

Добавьте следующую конфигурацию в `config/process.php`.

```php
return [
    // ... остальные настройки процессов опущены ...
    
    // websocket_test — имя процесса
    'websocket_test' => [
        // Здесь указывается класс процесса, то есть класс Pusher, определённый выше
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## Пример пользовательского процесса без прослушивания

Создайте `app/TaskTest.php`.

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // Каждые 10 секунд проверять базу данных на новые регистрации пользователей
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

Добавьте следующую конфигурацию в `config/process.php`.

```php
return [
    // ... остальные настройки процессов опущены ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> Примечание: при опускании listen процесс не прослушивает никаких портов. При опускании count количество процессов по умолчанию равно 1.

## Описание файла конфигурации

Полная конфигурация процесса определяется следующим образом:

```php
return [
    // ... 
    
    // websocket_test — имя процесса
    'websocket_test' => [
        // Здесь указывается класс процесса
        'handler' => app\Pusher::class,
        // Протокол, IP и порт для прослушивания (опционально)
        'listen'  => 'websocket://0.0.0.0:8888',
        // Количество процессов (опционально, по умолчанию 1)
        'count'   => 2,
        // Пользователь для выполнения процесса (опционально, по умолчанию текущий)
        'user'    => '',
        // Группа пользователей для выполнения процесса (опционально, по умолчанию текущая)
        'group'   => '',
        // Поддержка перезагрузки процесса (опционально, по умолчанию true)
        'reloadable' => true,
        // Включение reusePort
        'reusePort'  => true,
        // Транспорт (опционально, установить ssl при необходимости SSL, по умолчанию tcp)
        'transport'  => 'tcp',
        // Контекст (опционально, передать путь к сертификату при transport = ssl)
        'context'    => [], 
        // Параметры конструктора класса процесса (опционально)
        'constructor' => [],
        // Включён ли этот процесс
        'enable' => true
    ],
];
```

## Заключение

Пользовательские процессы в webman по сути представляют собой простую обёртку workerman. Они отделяют конфигурацию от бизнес-логики и реализуют колбэки `onXXX` workerman через методы класса. Все остальные аспекты использования совпадают с workerman.
