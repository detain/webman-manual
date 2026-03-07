# Обработка медленных операций

Иногда нам необходимо обрабатывать медленные операции. Чтобы избежать влияния медленных операций на обработку других запросов в webman, в зависимости от ситуации можно использовать разные решения.

## Решение 1: Использование очереди сообщений
См. [очередь Redis](../queue/redis.md) [очередь Stomp](../queue/stomp.md)

#### Преимущества
Может обрабатывать внезапные всплески запросов на обработку

#### Недостатки
Невозможно напрямую вернуть результат клиенту. При необходимости отправки результата требуется координация с другими сервисами, например использование [webman/push](https://www.workerman.net/plugin/2) для отправки результата обработки.

## Решение 2: Добавление нового HTTP-порта

Добавление нового HTTP-порта для обработки медленных запросов. Эти медленные запросы обрабатываются определённой группой процессов через доступ к этому порту, после обработки результат возвращается напрямую клиенту.

#### Преимущества
Может возвращать данные напрямую клиенту

#### Недостатки
Не может обрабатывать внезапные всплески запросов

#### Шаги реализации
Добавьте следующую конфигурацию в `config/process.php`.
```php
return [
    // ... Остальные настройки опущены ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Количество процессов
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Настройка класса запроса
            'logger' => \support\Log::channel('default'), // Экземпляр логгера
            'appPath' => app_path(), // Расположение каталога app
            'publicPath' => public_path() // Расположение каталога public
        ]
    ]
];
```

Таким образом, медленные интерфейсы могут обрабатываться группой процессов по адресу `http://127.0.0.1:8686/` без влияния на обработку других процессов.

Чтобы фронтенд не замечал разницу портов, можно добавить прокси к порту 8686 в nginx. Если пути медленных запросов начинаются с `/task`, конфигурация nginx будет выглядеть следующим образом:
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# Добавить новый upstream 8686
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # Запросы, начинающиеся с /task, направляются на порт 8686; при необходимости измените /task на нужный префикс
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # Остальные запросы направляются на порт 8787
  location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      if (!-f $request_filename){
          proxy_pass http://webman;
      }
  }
}
```

При обращении клиента к `домен.com/task/xxx` запрос будет обработан отдельным портом 8686 без влияния на обработку запросов порта 8787.

## Решение 3: Использование HTTP Chunked для асинхронной посылки сегментированных данных

#### Преимущества
Может возвращать данные напрямую клиенту

**Установка workerman/http-client**

```
composer require workerman/http-client
```

**app/controller/IndexController.php**
```php
<?php
namespace app\controller;

use support\Request;
use support\Response;
use Workerman\Protocols\Http\Chunk;

class IndexController
{
    public function index(Request $request)
    {
        $connection = $request->connection;
        $http = new \Workerman\Http\Client();
        $http->get('https://example.com/', function ($response) use ($connection) {
            $connection->send(new Chunk($response->getBody()));
            $connection->send(new Chunk('')); // Отправка пустого chunk означает конец ответа
        });
        // Сначала отправляются HTTP-заголовки, данные отправляются асинхронно
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **Примечание**
> В этом примере клиент `workerman/http-client` используется для асинхронного получения HTTP-результатов и возврата данных. Также можно использовать другие асинхронные клиенты, например [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html).
