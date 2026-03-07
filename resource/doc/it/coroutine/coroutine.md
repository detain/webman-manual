# Coroutine

webman è basato su Workerman, quindi webman può utilizzare le funzionalità di coroutine di Workerman.
Le coroutine supportano tre driver: `Swoole`, `Swow` e `Fiber`.

## Prerequisiti

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- Estensione Swoole o Swow installata, oppure `composer require revolt/event-loop` (per Fiber)
- Le coroutine sono disabilitate di default; vanno abilitate tramite l'impostazione `eventLoop`

## Come abilitare

webman supporta driver di coroutine diversi per processo. In `config/process.php` (inclusa la configurazione process.php dei plugin) si imposta tramite `eventLoop`:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // Vuoto di default, selezione automatica Select o Event, coroutine disabilitate
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
        // Per abilitare: Workerman\Events\Swoole::class, Workerman\Events\Swow::class o Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... altra configurazione omessa ...
];
```

> **Suggerimento**
> webman permette un `eventLoop` diverso per processo, quindi puoi abilitare le coroutine solo per processi specifici.
> Nell'esempio, la porta 8787 le ha disabilitate e la 8686 abilitate. Con nginx puoi fare un deploy misto con e senza coroutine.

## Esempio di coroutine

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

Con `eventLoop` impostato su `Swoole`, `Swow` o `Fiber`, webman crea una coroutine per richiesta e consente di crearne altre nel gestore.

## Limitazioni delle coroutine

* Con Swoole o Swow, in caso di I/O bloccante la coroutine passa automaticamente: il codice sincrono viene eseguito in modo asincrono.
* Con Fiber, l'I/O bloccante non provoca cambio di coroutine; il processo si blocca.
* Evitare che più coroutine operino sullo stesso ресурso (connessione DB, file, ecc.) senza protezione. Usare pool di connessioni o lock.
* Non memorizzare stato legato alla richiesta in variabili globali o statiche. Usare il contesto delle coroutine (`context`).

## Altre note

Swow intercetta le funzioni bloccanti di PHP a basso livello, alterando parte del comportamento standard. Se Swow è installato ma non usato, può causare bug.

**Raccomandazioni:**
* Se il progetto non usa Swow: non installare l'estensione Swow.
* Se usa Swow: impostare `eventLoop` a `Workerman\Events\Swow::class`.

## Contesto delle coroutine

Nell'ambiente delle coroutine, non memorizzare **stato legato alla richiesta** in variabili globali o statiche. Esempio errato:

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

> **Nota**
> Le variabili globali o statiche non sono vietate in generale; è vietato memorizzarvi **stato legato alla richiesta**.
> Configurazione globale, connessioni DB, singleton, ecc. possono stare in globali o statiche.

Con un solo processo e due richieste consecutive:
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

ci si aspetta `lilei` e `hanmeimei`, ma entrambe restituiscono `hanmeimei`. La seconda richiesta sovrascrive `$name`; al risveglio della prima il valore è già `hanmeimei`.

**Metodo corretto: memorizzare lo stato nel context**

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

La classe `support\Context` memorizza i dati del contesto delle coroutine. Al termine della coroutine vengono eliminati automaticamente.
Con le coroutine, ogni richiesta ha la propria coroutine, quindi il context viene ripulito alla fine della richiesta.
Senza coroutine, il context viene ripulito alla fine della richiesta.

**Le variabili locali non inquinano i dati**

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

Essendo `$name` una variabile locale, le coroutine non possono accedere alle variabili locali delle altre, quindi l'uso di variabili locali è sicuro in questo ambiente.

## Locker

Se un componente o una logica non è pensato per le coroutine, possono verificarsi conflitti di risorse o problemi di atomicità. In tal caso usare `Workerman\Locker` per serializzare l'accesso:

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
        // Senza lock, Swoole può lanciare errori come "Socket#10 has already been bound to another coroutine#10"
        // Swow può provocare coredump
        // Fiber: nessun problema, l'estensione Redis usa I/O bloccante
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel

Per eseguire più attività in parallelo e raccogliere i risultati, usare `Workerman\Parallel`:

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
                return $i;
            });
        }
        $results = $parallel->wait();
        return json($results); // Risposta: [1,2,3,4]
    }

}
```

## Pool (pool di connessioni)

Più coroutine che condividono la stessa connessione possono corrompere i dati. Usare un pool per DB, Redis, ecc.

webman offre già [webman/database](../db/tutorial.md), [webman/redis](../db/redis.md), [webman/cache](../db/cache.md), [webman/think-orm](../db/thinkorm.md) e [webman/think-cache](../db/thinkcache.md). Tutti includono pool e funzionano con e senza coroutine.

Per adattare un componente senza pool, usare `Workerman\Pool`. Esempio:

**Componente database**

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

**Utilizzo**

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

## Ulteriori informazioni sulle coroutine

Vedere la [documentazione Workerman sulle coroutine](https://www.workerman.net/doc/workerman/coroutine/coroutine.html).

## Deploy misto con e senza coroutine

webman permette di far coesistere servizi con e senza coroutine, ad esempio senza per il traffico normale e con per l'I/O lento, instradando il traffico tramite nginx.

Esempio `config/process.php`:

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
    
    // ... altra configurazione omessa ...
];
```

Instradamento nginx:

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
