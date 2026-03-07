# Coroutine

webman est construit sur Workerman, donc webman peut utiliser les fonctionnalités de coroutines de Workerman.
Les coroutines supportent trois pilotes : `Swoole`, `Swow` et `Fiber`.

## Prérequis

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- Extension Swoole ou Swow installée, ou `composer require revolt/event-loop` (pour Fiber)
- Les coroutines sont désactivées par défaut ; elles doivent être activées via le paramètre `eventLoop`

## Activer les coroutines

webman permet de configurer différents pilotes de coroutines par processus. Dans `config/process.php` (y compris la config process.php des plugins), configurez via `eventLoop` :

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // Vide par défaut, sélection automatique Select ou Event, coroutines désactivées
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
        // Pour activer : Workerman\Events\Swoole::class, Workerman\Events\Swow::class ou Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... autre config omise ...
];
```

> **Conseil**
> webman permet un `eventLoop` différent par processus, donc vous pouvez activer les coroutines uniquement pour certains processus.
> Dans l’exemple, le port 8787 les a désactivées et le 8686 activées. Avec nginx, vous pouvez déployer un mélange de services avec et sans coroutines.

## Exemple de coroutine

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

Avec `eventLoop` égal à `Swoole`, `Swow` ou `Fiber`, webman crée une coroutine par requête et permet d’en créer d’autres dans le handler.

## Limites des coroutines

* Avec Swoole ou Swow, en cas d’I/O bloquant, la coroutine bascule automatiquement : le code synchrone s’exécute de façon asynchrone.
* Avec Fiber, l’I/O bloquant ne provoque pas de bascule ; le processus bloque.
* Ne laissez pas plusieurs coroutines accéder au même ressource (connexion BD, fichier, etc.) sans protection. Utilisez des pools ou des verrous.
* Ne stockez pas d’état lié à la requête dans des variables globales ou statiques. Utilisez le contexte de coroutine (`context`).

## Autres points

Swow intercepte les fonctions bloquantes de PHP à bas niveau, ce qui modifie une partie du comportement PHP. Si Swow est installé mais non utilisé, cela peut provoquer des bugs.

**Recommandations :**
* Si le projet n’utilise pas Swow : ne pas installer l’extension Swow.
* S’il utilise Swow : définir `eventLoop` à `Workerman\Events\Swow::class`.

## Contexte des coroutines

Dans l’environnement de coroutines, ne stockez pas d’**état lié à la requête** dans des variables globales ou statiques. Exemple incorrect :

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

> **Note**
> Les variables globales ou statiques ne sont pas interdites en soi ; ce qui est interdit est d’y stocker **l’état lié à la requête**.
> Configuration globale, connexions BD, singletons, etc. peuvent être stockés dans des globales ou statiques.

Avec un seul processus et deux requêtes consécutives :
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

on s’attend à `lilei` et `hanmeimei`, mais les deux renvoient `hanmeimei`. La deuxième requête écrase `$name` ; au réveil de la première, la valeur est déjà `hanmeimei`.

**La bonne méthode : stocker l’état dans le context**

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

La classe `support\Context` stocke les données de contexte des coroutines. À la fin de la coroutine, ces données sont supprimées automatiquement.
Avec les coroutines, chaque requête a sa propre coroutine, donc le context est nettoyé à la fin de la requête.
Sans coroutines, le context est nettoyé à la fin de la requête.

**Les variables locales ne polluent pas les données**

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

Comme `$name` est une variable locale, les coroutines ne peuvent pas accéder aux variables locales des autres, donc les variables locales sont sûres dans cet environnement.

## Locker

Si un composant ou une logique n’a pas été conçu pour les coroutines, des conflits de ressources ou des problèmes d’atomicité peuvent survenir. Dans ce cas, utiliser `Workerman\Locker` pour sérialiser l’accès :

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
        // Sans verrou : Swoole peut lever des erreurs comme "Socket#10 has already been bound to another coroutine#10"
        // Swow peut provoquer un coredump
        // Fiber : pas de problème car l’extension Redis utilise un I/O bloquant
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel

Pour exécuter plusieurs tâches en parallèle et récupérer les résultats, utiliser `Workerman\Parallel` :

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
        return json($results); // Réponse : [1,2,3,4]
    }

}
```

## Pool (pool de connexions)

Plusieurs coroutines partageant la même connexion peuvent corrompre les données. Utiliser un pool pour la BD, Redis, etc.

webman fournit déjà [webman/database](../db/tutorial.md), [webman/redis](../db/redis.md), [webman/cache](../db/cache.md), [webman/think-orm](../db/thinkorm.md) et [webman/think-cache](../db/thinkcache.md). Tous intègrent des pools et fonctionnent avec et sans coroutines.

Pour adapter un composant sans pool, utiliser `Workerman\Pool`. Exemple :

**Composant base de données**

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

**Utilisation**

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

## En savoir plus sur les coroutines

Voir la [documentation Workerman sur les coroutines](https://www.workerman.net/doc/workerman/coroutine/coroutine.html).

## Déploiement mixte avec et sans coroutines

webman permet de faire coexister des services avec et sans coroutines, par exemple sans coroutines pour le trafic normal et avec coroutines pour l’I/O lent, en routant le trafic avec nginx.

Exemple `config/process.php` :

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
    
    // ... autre config omise ...
];
```

Routage nginx :

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
