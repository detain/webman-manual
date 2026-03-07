# Очередь Redis

Очередь сообщений на основе Redis с поддержкой отложенной обработки.

## Установка
`composer require webman/redis-queue`

## Файл конфигурации
Файл конфигурации Redis создаётся автоматически в `{основной-проект}/config/plugin/webman/redis-queue/redis.php`, содержимое примерно следующее:
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // Пароль, необязательно
            'db' => 0,            // База данных
            'max_attempts'  => 5, // Количество повторных попыток после неудачного потребления
            'retry_seconds' => 5, // Интервал между попытками (секунды)
        ]
    ],
];
```

### Повтор при сбое потребления
При сбое потребления (исключении) сообщение помещается в отложенную очередь и ждёт следующей попытки. Количество попыток задаётся `max_attempts`, интервал — `retry_seconds` и `max_attempts`. Например, при `max_attempts`=5 и `retry_seconds`=10 интервал 1-й попытки — `1*10` с, 2-й — `2*10` с, 3-й — `3*10` с и т.д. до 5 попыток. При превышении числа попыток из `max_attempts` сообщение попадает в очередь сбоев с ключом `{redis-queue}-failed`.

## Отправка сообщений (синхронно)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // Имя очереди
        $queue = 'send-mail';
        // Данные, можно передать массив напрямую, без сериализации
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Отправить сообщение
        Redis::send($queue, $data);
        // Отправить отложенное сообщение, обработается через 60 секунд
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
При успешной отправке `Redis::send()` возвращает true, иначе false или исключение.

> **Подсказка**
> Время потребления отложенной очереди может отличаться. Например, если потребление медленнее производства, очередь накапливается и потребление задерживается. Решение: запустить больше процессов потребления.

## Отправка сообщений (асинхронно)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // Имя очереди
        $queue = 'send-mail';
        // Данные, можно передать массив напрямую, без сериализации
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Отправить сообщение
        Client::send($queue, $data);
        // Отправить отложенное сообщение, обработается через 60 секунд
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` не возвращает значение. Это асинхронная отправка, не гарантирующая 100% доставки в Redis.

> **Подсказка**
> Принцип `Client::send()` — создание очереди в локальной памяти и асинхронная синхронизация сообщений в Redis (синхронизация быстрая, ~10 000 сообщений/с). При перезапуске процесса до полной синхронизации данные могут теряться. Асинхронная отправка `Client::send()` подходит для некритичных сообщений.

> **Подсказка**
> `Client::send()` асинхронен и может использоваться только в среде Workerman. Для скриптов командной строки используйте синхронный интерфейс `Redis::send()`.

## Отправка сообщений из других проектов
Иногда нужно отправлять сообщения из других проектов, где нельзя использовать `webman\redis-queue`. В таких случаях можно использовать следующую функцию для отправки в очередь:

```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting';
    $queue_delay = '{redis-queue}-delayed';
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```

Параметр `$redis` — экземпляр Redis. Пример использования расширения redis:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## Потребление
Файл конфигурации процесса потребления: `{основной-проект}/config/plugin/webman/redis-queue/process.php`. Каталог потребителей: `{основной-проект}/app/queue/redis/`.

Команда `php webman redis-queue:consumer my-send-mail` создаёт файл `{основной-проект}/app/queue/redis/MyMailSend.php`.

> **Подсказка**
> Для этой команды нужен плагин [Консоль](../plugin/console.md). Если не устанавливать его, можно создать файл вручную, например так:

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // Имя очереди для потребления
    public $queue = 'send-mail';

    // Имя подключения, соответствует подключению в plugin/webman/redis-queue/redis.php
    public $connection = 'default';

    // Потребление
    public function consume($data)
    {
        // Десериализация не нужна
        var_export($data); // Выведет ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // Обработчик сбоя потребления
    /* 
    $package = [
        'id' => 1357277951, // ID сообщения
        'time' => 1709170510, // Время сообщения
        'delay' => 0, // Время задержки
        'attempts' => 2, // Количество потреблений
        'queue' => 'send-mail', // Имя очереди
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // Содержимое сообщения
        'max_attempts' => 5, // Макс. количество попыток
        'error' => 'Сообщение об ошибке' // Сообщение об ошибке
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // Десериализация не нужна
        var_export($package); 
    }
}
```

> **Примечание**
> Потребление считается успешным, если во время него не выброшено исключение или Error; иначе — сбой, сообщение попадает в очередь повторных попыток. redis-queue не имеет механизма ack; можно считать его автоматическим ack (если нет исключения или Error). Чтобы пометить текущее сообщение как потреблённое неудачно, выбросьте исключение вручную, чтобы отправить его в очередь повторных попыток. По сути это то же, что механизм ack.

> **Подсказка**
> Потребители поддерживают несколько серверов и процессов; одно сообщение **не** будет потреблено дважды. Потреблённые сообщения автоматически удаляются из очереди; удаление вручную не требуется.

> **Подсказка**
> Процессы потребления могут обрабатывать несколько разных очередей одновременно. Добавление новой очереди не требует изменения конфигурации в `process.php`. При добавлении потребителя новой очереди достаточно добавить класс `Consumer` в `app/queue/redis` и указать имя очереди свойством `$queue`.

> **Подсказка**
> Пользователям Windows нужно запускать `php windows.php` для старта webman, иначе процесс потребления не запустится.

> **Подсказка**
> Колбэк onConsumeFailure вызывается при каждом сбое потребления. Здесь можно обработать логику после сбоя. (Функция требует `webman/redis-queue>=1.3.2` и `workerman/redis-queue>=1.2.1`)

## Разные процессы потребления для разных очередей
По умолчанию все потребители используют один процесс потребления. Иногда нужно разделить потребление для части очередей — например, медленный бизнес в одной группе процессов, быстрый — в другой. Для этого потребителей можно разнести по двум каталогам: `app_path() . '/queue/redis/fast'` и `app_path() . '/queue/redis/slow'` (нужно обновить namespace класса потребителя). Конфигурация:

```php
return [
    ...остальная конфигурация пропущена...
    
    'redis_consumer_fast'  => [ // Ключ произвольный, ограничений по формату нет, здесь redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Каталог классов потребителей
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // Ключ произвольный, ограничений по формату нет, здесь redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Каталог классов потребителей
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

Быстрые потребители идут в `queue/redis/fast`, медленные — в `queue/redis/slow`, что позволяет назначать процессы потребления по очередям.

## Настройка нескольких Redis
#### Конфигурация
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // Пароль, строка, опционально
            'db' => 0,            // База данных
            'max_attempts'  => 5, // Повторы после сбоя потребления
            'retry_seconds' => 5, // Интервал (секунды)
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // Пароль, строка, опционально
            'db' => 0,            // База данных
            'max_attempts'  => 5, // Повторы после сбоя потребления
            'retry_seconds' => 5, // Интервал (секунды)
        ]
    ],
];
```

В конфигурацию добавлена Redis-настройка с ключом `other`.

#### Отправка сообщений в несколько Redis

```php
// Отправить в очередь с ключом `default`
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// То же, что
Client::send($queue, $data);
Redis::send($queue, $data);

// Отправить в очередь с ключом `other`
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### Потребление из нескольких Redis
Потребление из очереди с ключом `other` в конфигурации:
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // Имя очереди для потребления
    public $queue = 'send-mail';

    // === Установить 'other' здесь для потребления из очереди с ключом 'other' в конфигурации ===
    public $connection = 'other';

    // Потребление
    public function consume($data)
    {
        // Десериализация не нужна
        var_export($data);
    }
}
```

## Часто задаваемые вопросы

**Почему возникает ошибка `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)`?**

Эта ошибка возникает только при асинхронной отправке через `Client::send()`. Асинхронная отправка сначала сохраняет сообщения в локальную память, затем отправляет их в Redis при простое процесса. Если Redis принимает медленнее, чем идёт производство, или процесс занят другими задачами и не успевает синхронизировать сообщения из памяти в Redis, возникает накопление. При накоплении более 600 секунд запускается эта ошибка.

Решение: используйте синхронный интерфейс отправки `Redis::send()`.
