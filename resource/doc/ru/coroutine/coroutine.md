# Корутина

webman построен на Workerman, поэтому webman может использовать возможности корутин Workerman.
Поддерживаются три драйвера корутин: `Swoole`, `Swow` и `Fiber`.

## Требования

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- Установленное расширение Swoole или Swow, либо `composer require revolt/event-loop` (для Fiber)
- Корутины по умолчанию отключены, их нужно включать отдельно через настройку `eventLoop`

## Как включить

webman позволяет задавать разные драйверы корутин для разных процессов. Настройка выполняется в `config/process.php` (включая `process.php` плагинов) через опцию `eventLoop`:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // По умолчанию пусто — авто выбор Select или Event, корутины отключены
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ],
    'my-coroutine' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        // Для включения корутин: Workerman\Events\Swoole::class, Workerman\Events\Swow::class или Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... остальная конфигурация опущена ...
];
```

> **Подсказка**
> webman позволяет задать разный `eventLoop` для разных процессов, то есть корутины можно включать только для нужных процессов.
> В примере выше для порта 8787 корутины выключены, для порта 8686 — включены. В сочетании с nginx можно развернуть смесь корутинных и некорутинных сервисов.

## Пример корутины

```php
<?php
namespace app\controller;

use support\Response;
use Workerman\Coroutine;
use Workerman\Timer;

class IndexController
{
    public function index(): Response
    {
        Coroutine::create(function(){
            Timer::sleep(1.5);
            echo "hello coroutine\n";
        });
        return response('hello webman');
    }

}
```

При `eventLoop` равном `Swoole`, `Swow` или `Fiber` webman создаёт корутину на каждый запрос. В обработчике можно создавать дополнительные корутины для бизнес-логики.

## Ограничения корутин

* При Swoole или Swow при блокирующем I/O корутина автоматически переключается — синхронный код выполняется асинхронно.
* При драйвере Fiber блокирующее I/O не вызывает переключение корутин, процесс просто блокируется.
* Не допускайте одновременной работы нескольких корутин с одним и тем же ресурсом (подключение к БД, файл и т.п.) без защиты — возможна конкуренция. Используйте пулы подключений или блокировки.
* Не храните состояние, связанное с запросом, в глобальных или статических переменных — возможна глобальная «загрязнённость». Используйте контекст корутины (`context`).

## Дополнительные замечания

Swow перехватывает блокирующие функции PHP, что меняет часть стандартного поведения. Если Swow установлен, но не используется, это может приводить к сбоям.

**Рекомендации:**
* Если проект не использует Swow, не устанавливайте расширение Swow.
* Если проект использует Swow, задайте `eventLoop` в `Workerman\Events\Swow::class`.

## Контекст корутины

В среде корутин не храните **связанное с запросом** состояние в глобальных или статических переменных. Пример неправильного подхода:

```php
<?php

namespace app\controller;

use support\Request;
use Workerman\Timer;

class TestController
{
    protected static $name = '';

    public function index(Request $request)
    {
        static::$name = $request->get('name');
        Timer::sleep(5);
        return static::$name;
    }
}
```

> **Важно**
> Глобальные и статические переменные сами по себе не запрещены — запрещено хранить в них **состояние, связанное с запросом**.
> Глобальная конфигурация, подключения к БД, синглтоны и т.п. можно хранить в глобальных/статических переменных.

При количестве процессов, равном 1, если отправить два запроса подряд:
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

ожидаемые ответы — `lilei` и `hanmeimei`, но фактически оба вернут `hanmeimei`. Второй запрос перезаписывает статическую переменную `$name`, и к моменту пробуждения первого запроса там уже `hanmeimei`.

**Правильно — хранить состояние запроса в context:**

```php
<?php

namespace app\controller;

use support\Request;
use support\Context;
use Workerman\Timer;

class TestController
{
    public function index(Request $request)
    {
        Context::set('name', $request->get('name'));
        Timer::sleep(5);
        return Context::get('name');
    }
}
```

Класс `support\Context` хранит контекстные данные корутины. По завершении корутины соответствующие данные удаляются автоматически.
При включённых корутинах каждый запрос выполняется в своей корутине, поэтому context очищается по завершении запроса.
Без корутин context очищается при завершении запроса.

**Локальные переменные не приводят к «загрязнению» данных**

```php
<?php

namespace app\controller;

use support\Request;
use support\Context;
use Workerman\Timer;

class TestController
{
    public function index(Request $request)
    {
        $name = $request->get('name');
        Timer::sleep(5);
        return $name;
    }
}
```

Локальные переменные недоступны между корутинами, поэтому их использование безопасно в среде корутин.

## Locker (блокировка)

Если какой-то компонент или логика не рассчитаны на корутины, возможны конкуренция за ресурсы или проблемы атомарности. В таких случаях можно использовать `Workerman\Locker` для сериализации доступа и избежания конкурентных проблем.

```php
<?php

namespace app\controller;

use Redis;
use support\Response;
use Workerman\Coroutine\Locker;

class IndexController
{
    public function index(): Response
    {
        static $redis;
        if (!$redis) {
            $redis = new Redis();
            $redis->connect('127.0.0.1', 6379);
        }
        // Без блокировки в Swoole возможны ошибки вроде "Socket#10 has already been bound to another coroutine#10"
        // В Swow возможен coredump
        // Fiber не страдает, так как расширение Redis использует блокирующий I/O
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel (параллельное выполнение)

Для параллельного выполнения нескольких задач и сбора результатов используйте `Workerman\Parallel`:

```php
<?php

namespace app\controller;

use support\Response;
use Workerman\Coroutine\Parallel;

class IndexController
{
    public function index(): Response
    {
        $parallel = new Parallel();
        for ($i=1; $i<5; $i++) {
            $parallel->add(function () use ($i) {
                // Выполнить что-либо
                return $i;
            });
        }
        $results = $parallel->wait();
        return json($results); // Ответ: [1,2,3,4]
    }

}
```

## Pool (пул подключений)

Совместное использование одного подключения несколькими корутинами может привести к искажению данных. Используйте пул подключений для БД, Redis и т.п.

webman уже предоставляет [webman/database](../db/tutorial.md), [webman/redis](../db/redis.md), [webman/cache](../db/cache.md), [webman/think-orm](../db/thinkorm.md), [webman/think-cache](../db/thinkcache.md). Все они используют пулы подключений и работают в корутинном и некорутинном режимах.

Чтобы адаптировать компонент без пула, используйте `Workerman\Pool`. Пример:

**Компонент базы данных**

```php
<?php
namespace app;

use Workerman\Coroutine\Context;
use Workerman\Coroutine;
use Workerman\Coroutine\Pool;

class Db
{
    private static ?Pool $pool = null;

    public static function __callStatic($name, $arguments)
    {
        if (self::$pool === null) {
            self::initializePool();
        }
        $pdo = Context::get('pdo');
        if (!$pdo) {
            $pdo = self::$pool->get();
            Context::set('pdo', $pdo);
            Coroutine::defer(function () use ($pdo) {
                self::$pool->put($pdo);
            });
        }
        return call_user_func_array([$pdo, $name], $arguments);
    }

    private static function initializePool(): void
    {
        self::$pool = new Pool(10);
        self::$pool->setConnectionCreator(function () {
            return new \PDO('mysql:host=127.0.0.1;dbname=your_database', 'your_username', 'your_password');
        });
        self::$pool->setConnectionCloser(function ($pdo) {
            $pdo = null;
        });
        self::$pool->setHeartbeatChecker(function ($pdo) {
            $pdo->query('SELECT 1');
        });
    }

}
```

**Использование**

```php
<?php
namespace app\controller;

use support\Response;
use app\Db;

class IndexController
{
    public function index(): Response
    {
        $value = Db::query('SELECT NOW() as now')->fetchAll();
        return json($value); // [{"now":"2025-02-06 23:41:03","0":"2025-02-06 23:41:03"}]
    }

}
```

## Подробнее о корутинах и связанных компонентах

См. [документацию Workerman по корутинам](https://www.workerman.net/doc/workerman/coroutine/coroutine.html).

## Смешанное развёртывание с корутинами и без

webman поддерживает совместную работу корутинных и некорутинных сервисов. Например, некорутины для обычных запросов и корутины для медленного I/O, с маршрутизацией через nginx.

Пример `config/process.php`:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '',
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ],
    'my-coroutine' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ],
    
    // ... остальная конфигурация опущена ...
];
```

Маршрутизация в nginx:

```nginx
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  location /tast {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

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
