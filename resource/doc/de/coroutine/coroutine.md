# Koroutine

webman basiert auf Workerman, daher kann webman die Koroutinen-Funktionen von Workerman nutzen.
Koroutinen unterstützen drei Treiber: `Swoole`, `Swow` und `Fiber`.

## Voraussetzungen

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- Swoole- oder Swow-Erweiterung installiert oder `composer require revolt/event-loop` (für Fiber)
- Koroutinen sind standardmäßig deaktiviert und müssen über `eventLoop` separat aktiviert werden

## Aktivierung

webman unterstützt unterschiedliche Koroutinen-Treiber pro Prozess. In `config/process.php` (inkl. Plugin-process.php-Konfiguration) kann der Treiber über `eventLoop` gesetzt werden:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // Standardmäßig leer, automatische Wahl von Select oder Event, Koroutinen deaktiviert
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
        // Koroutinen: Workerman\Events\Swoole::class, Workerman\Events\Swow::class oder Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... weitere Konfiguration ausgelassen ...
];
```

> **Hinweis**
> webman erlaubt pro Prozess einen anderen `eventLoop`. So können Koroutinen gezielt für bestimmte Prozesse aktiviert werden.
> Im Beispiel oben: Port 8787 ohne Koroutinen, Port 8686 mit Koroutinen. Mit nginx-Routing lässt sich ein Mix aus Koroutinen- und Nicht-Koroutinen-Services betreiben.

## Koroutinen-Beispiel

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

Bei `eventLoop` `Swoole`, `Swow` oder `Fiber` erstellt webman pro Request eine Koroutine und ermöglicht die Erstellung weiterer Koroutinen im Request-Handler.

## Einschränkungen von Koroutinen

* Mit Swoole oder Swow wechselt die Koroutine bei blockierendem I/O automatisch – synchroner Code läuft asynchron.
* Mit dem Fiber-Treiber führt blockierendes I/O nicht zum Wechsel; der Prozess blockiert.
* Mehrere Koroutinen sollten nicht gleichzeitig dieselbe Ressource (DB-Verbindung, Datei usw.) nutzen – Risiko von Ressourcenkonflikten. Verbindungspools oder Locks verwenden.
* Anfragebezogene Zustände nicht in globalen oder statischen Variablen speichern – Risiko von globalem Datenkonflikt. Stattdessen den Koroutinen-Kontext (`context`) nutzen.

## Weitere Hinweise

Swow hängt PHP-blockierende Funktionen auf niedriger Ebene. Das ändert Teile des Standardverhaltens von PHP. Wenn Swow installiert, aber nicht genutzt wird, kann das zu Fehlern führen.

**Empfehlungen:**
* Wenn das Projekt Swow nicht nutzt: Swow-Erweiterung nicht installieren.
* Wenn Swow genutzt wird: `eventLoop` auf `Workerman\Events\Swow::class` setzen.

## Koroutinen-Kontext

Im Koroutinen-Umfeld **anfragebezogene** Zustände nicht in globalen oder statischen Variablen speichern. Beispiel falscher Ansatz:

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

> **Hinweis**
> Globale oder statische Variablen sind nicht generell verboten – verboten ist das Speichern **anfragebezogener Zustände** darin.
> Globale Konfiguration, DB-Verbindungen, Singletons usw. dürfen in globalen/statischen Variablen gehalten werden.

Bei Prozessanzahl 1 und zwei aufeinanderfolgenden Anfragen:
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

erwartet man `lilei` und `hanmeimei`, tatsächlich kommt bei beiden `hanmeimei`. Die zweite Anfrage überschreibt `$name`, beim Aufwachen der ersten ist der Wert bereits `hanmeimei`.

**Korrekt: Zustand im Context speichern**

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

`support\Context` speichert Koroutinen-Kontext. Nach Abschluss der Koroutine werden die Daten automatisch gelöscht.
Mit Koroutinen läuft jeder Request in eigener Koroutine – der Context wird nach dem Request automatisch gelöscht.
Ohne Koroutinen wird der Context am Request-Ende gelöscht.

**Lokale Variablen sind unkritisch**

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

`$name` ist eine lokale Variable, die von anderen Koroutinen nicht zugreifbar ist – daher koroutinen-sicher.

## Locker

Wenn eine Komponente oder Logik nicht für Koroutinen ausgelegt ist, können Ressourcenkonflikte oder Atomizitätsprobleme auftreten. In solchen Fällen kann `Workerman\Locker` zur Serialisierung des Zugriffs verwendet werden:

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
        // Ohne Lock: Swoole kann Fehler wie "Socket#10 has already been bound to another coroutine#10" werfen
        // Swow kann zu Coredump führen
        // Fiber: unkritisch, da Redis-Erweiterung blockierend arbeitet
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel

Für parallele Ausführung mehrerer Aufgaben und Sammlung der Ergebnisse: `Workerman\Parallel` verwenden:

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
        return json($results); // Antwort: [1,2,3,4]
    }

}
```

## Pool (Verbindungspool)

Mehrere Koroutinen, die dieselbe Verbindung teilen, können Daten mischen. Daher Verbindungspools für DB, Redis usw. verwenden.

webman bietet bereits [webman/database](../db/tutorial.md), [webman/redis](../db/redis.md), [webman/cache](../db/cache.md), [webman/think-orm](../db/thinkorm.md), [webman/think-cache](../db/thinkcache.md). Alle haben Pools und funktionieren mit und ohne Koroutinen.

Für Komponenten ohne Pool kann `Workerman\Pool` verwendet werden. Beispiel:

**Datenbank-Komponente**

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

**Verwendung**

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

## Weitere Infos zu Koroutinen

Siehe [Workerman-Koroutinen-Dokumentation](https://www.workerman.net/doc/workerman/coroutine/coroutine.html).

## Gemeinsamer Betrieb mit und ohne Koroutinen

webman unterstützt einen Mix aus Koroutinen- und Nicht-Koroutinen-Services, z. B. normale Requests ohne Koroutinen, langsame I/O-Requests mit Koroutinen, Routing über nginx.

Beispiel `config/process.php`:

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
    
    // ... weitere Konfiguration ausgelassen ...
];
```

nginx-Routing:

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
