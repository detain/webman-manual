# Очередь Stomp

Stomp — это простой текстовый протокол направленной передачи сообщений, обеспечивающий формат связи, позволяющий клиентам Stomp взаимодействовать с любым посредником сообщений Stomp (брокером). [workerman/stomp](https://github.com/walkor/stomp) реализует клиент Stomp, используемый в основном для сценариев очередей сообщений, таких как RabbitMQ, Apollo, ActiveMQ и т. д.

## Установка
`composer require webman/stomp`

## Конфигурация
Файл конфигурации находится в `config/plugin/webman/stomp`

## Отправка сообщений
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // Очередь
        $queue = 'examples';
        // Данные (при передаче массива необходима ручная сериализация, напр. с помощью json_encode, serialize и т. д.)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // Выполнить отправку
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> Для совместимости с другими проектами компонент Stomp не предоставляет автоматическую сериализацию и десериализацию. При отправке массива необходима ручная сериализация и десериализация при потреблении.

## Потребление сообщений
Создайте файл `app/queue/stomp/MyMailSend.php` (имя класса может быть любым при соблюдении стандарта PSR-4).
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // Имя очереди
    public $queue = 'examples';

    // Имя подключения, соответствующее подключению в stomp.php
    public $connection = 'default';

    // При значении 'client' необходимо вызвать $ack_resolver->ack() для уведомления сервера об успешном потреблении
    // При значении 'auto' вызов $ack_resolver->ack() не требуется
    public $ack = 'auto';

    // Потребление
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // Если данные — массив, необходима ручная десериализация
        var_export(json_decode($data, true)); // выводит ['to' => 'tom@gmail.com', 'content' => 'hello']
        // Уведомить сервер об успешном потреблении
        $ack_resolver->ack(); // при ack 'auto' этот вызов можно опустить
    }
}
```

# Включение протокола Stomp в RabbitMQ
По умолчанию RabbitMQ не включает протокол Stomp. Выполните следующую команду для включения:
```
rabbitmq-plugins enable rabbitmq_stomp
```
После включения порт Stomp по умолчанию — 61613.
