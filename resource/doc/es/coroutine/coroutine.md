# Corutina

webman está construido sobre Workerman, por lo que puede utilizar las funcionalidades de corutinas de Workerman.
Se soportan tres controladores: `Swoole`, `Swow` y `Fiber`.

## Requisitos previos

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- Extensión Swoole o Swow instalada, o `composer require revolt/event-loop` (para Fiber)
- Las corutinas vienen desactivadas por defecto; deben activarse mediante la opción `eventLoop`

## Cómo activar

webman permite configurar distintos controladores de corutinas por proceso. En `config/process.php` (incluida la configuración process.php de plugins) se configura mediante `eventLoop`:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // Vacío por defecto, elige Select o Event automáticamente, corutinas desactivadas
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
        // Para activar corutinas: Workerman\Events\Swoole::class, Workerman\Events\Swow::class o Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... resto de configuración omitida ...
];
```

> **Consejo**
> webman permite un `eventLoop` distinto por proceso, así que puedes activar corutinas solo para procesos concretos.
> En el ejemplo, el puerto 8787 las tiene desactivadas y el 8686 activadas. Con nginx puedes desplegar una combinación de servicios con y sin corutinas.

## Ejemplo de corutina

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

Con `eventLoop` como `Swoole`, `Swow` o `Fiber`, webman crea una corutina por solicitud y permite crear más dentro del manejador.

## Limitaciones de las corutinas

* Con Swoole o Swow, ante I/O bloqueante la corutina cambia automáticamente y el código síncrono se ejecuta de forma asíncrona.
* Con Fiber, el I/O bloqueante no provoca cambio de corutina; el proceso se bloquea.
* No uses varias corutinas sobre el mismo recurso (conexión a BD, archivo, etc.) sin protegerlo. Usa pools de conexiones o bloqueos.
* No almacenes estado vinculado a la solicitud en variables globales o estáticas. Usa el contexto de corutina (`context`).

## Otras consideraciones

Swow intercepta funciones bloqueantes de PHP a bajo nivel, lo que puede alterar el comportamiento estándar. Si Swow está instalado pero no se usa, puede causar errores.

**Recomendaciones:**
* Si el proyecto no usa Swow, no instalar la extensión Swow.
* Si usa Swow, configurar `eventLoop` como `Workerman\Events\Swow::class`.

## Contexto de corutina

No guardes **estado ligado a la solicitud** en variables globales o estáticas dentro del entorno de corutinas. Ejemplo incorrecto:

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
> Las variables globales o estáticas no están prohibidas en general; sí está prohibido almacenar en ellas **estado vinculado a la solicitud**.
> Configuración global, conexiones a BD, singletons, etc. sí pueden guardarse en globales o estáticas.

Con un solo proceso y dos solicitudes seguidas:
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

se esperarían `lilei` y `hanmeimei`, pero ambas devuelven `hanmeimei`. La segunda sobrescribe la variable estática `$name` y, al despertar la primera, el valor ya es `hanmeimei`.

**Forma correcta: almacenar el estado en el context**

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

La clase `support\Context` guarda datos del contexto de corutinas. Al finalizar la corutina se eliminan automáticamente.
Con corutinas activas, cada solicitud usa su propia corutina, por lo que el context se limpia al terminar la solicitud.
Sin corutinas, el context se limpia al final de la solicitud.

**Las variables locales no contaminan los datos**

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

Al ser `$name` una variable local, las corutinas no pueden acceder entre sí a sus variables locales, por lo que el uso de variables locales es seguro en este entorno.

## Locker

Si un componente o lógica no contemplan corutinas, pueden surgir conflictos de recursos o problemas de atomicidad. En esos casos usar `Workerman\Locker` para serializar el acceso:

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
        // Sin bloqueo, Swoole puede lanzar errores como "Socket#10 has already been bound to another coroutine#10"
        // Swow puede provocar coredump
        // Con Fiber no hay problema porque la extensión Redis usa I/O bloqueante
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel (ejecución paralela)

Para ejecutar varias tareas en paralelo y reunir resultados, usar `Workerman\Parallel`:

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
        return json($results); // Respuesta: [1,2,3,4]
    }

}
```

## Pool (agrupación de conexiones)

Varias corutinas compartiendo una misma conexión pueden corromper datos. Usar pools para bases de datos, Redis, etc.

webman ya ofrece [webman/database](../db/tutorial.md), [webman/redis](../db/redis.md), [webman/cache](../db/cache.md), [webman/think-orm](../db/thinkorm.md) y [webman/think-cache](../db/thinkcache.md). Todas incluyen pools y funcionan con y sin corutinas.

Para adaptar un componente sin pool, usar `Workerman\Pool`. Ejemplo:

**Componente de base de datos**

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

**Uso**

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

## Más sobre corutinas

Consultar la [documentación de corutinas de Workerman](https://www.workerman.net/doc/workerman/coroutine/coroutine.html).

## Despliegue mixto con y sin corutinas

webman permite combinar servicios con y sin corutinas, por ejemplo sin corutinas para tráfico normal y con corutinas para I/O lento, encaminando el tráfico con nginx.

Ejemplo en `config/process.php`:

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
    
    // ... resto de configuración omitida ...
];
```

Encaminamiento en nginx:

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
