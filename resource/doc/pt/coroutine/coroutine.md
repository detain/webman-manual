# Coroutine

O webman foi construído sobre o Workerman, por isso pode utilizar as funcionalidades de coroutines do Workerman.
As coroutines suportam três drivers: `Swoole`, `Swow` e `Fiber`.

## Requisitos prévios

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- Extensão Swoole ou Swow instalada, ou `composer require revolt/event-loop` (para Fiber)
- As coroutines vêm desativadas por defeito; devem ser ativadas via a opção `eventLoop`

## Como ativar

O webman permite configurar drivers de coroutines diferentes por processo. Em `config/process.php` (incluindo a configuração process.php dos plugins), configurar via `eventLoop`:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // Vazio por defeito, escolhe Select ou Event automaticamente, coroutines desativadas
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
        // Para ativar: Workerman\Events\Swoole::class, Workerman\Events\Swow::class ou Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... resto da configuração omitido ...
];
```

> **Dica**
> O webman permite um `eventLoop` diferente por processo, podendo ativar coroutines apenas para processos específicos.
> No exemplo, a porta 8787 tem coroutines desativadas e a 8686 ativadas. Com nginx é possível fazer um deploy misto com e sem coroutines.

## Exemplo de coroutine

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

Com `eventLoop` como `Swoole`, `Swow` ou `Fiber`, o webman cria uma coroutine por pedido e permite criar mais dentro do handler.

## Limitações das coroutines

* Com Swoole ou Swow, perante I/O bloqueante a coroutine troca automaticamente e o código síncrono corre de forma assíncrona.
* Com Fiber, o I/O bloqueante não provoca troca de coroutine; o processo bloqueia.
* Evitar várias coroutines a usar o mesmo recurso (ligação BD, ficheiro, etc.) sem proteção. Usar pools de ligações ou bloqueios.
* Não armazenar estado ligado ao pedido em variáveis globais ou estáticas. Usar o contexto de coroutine (`context`).

## Outras considerações

O Swow intercepta funções bloqueantes do PHP a baixo nível, o que pode alterar o comportamento padrão. Se o Swow está instalado mas não é usado, pode causar erros.

**Recomendações:**
* Se o projeto não usa Swow: não instalar a extensão Swow.
* Se usa Swow: configurar `eventLoop` como `Workerman\Events\Swow::class`.

## Contexto da coroutine

Não armazenar **estado ligado ao pedido** em variáveis globais ou estáticas no ambiente de coroutines. Exemplo incorreto:

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
> Variáveis globais ou estáticas não estão proibidas em geral; o que está proibido é armazenar nelas **estado ligado ao pedido**.
> Configuração global, ligações BD, singletons, etc. podem ser armazenados em globais ou estáticas.

Com um único processo e dois pedidos consecutivos:
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

esperar-se-iam `lilei` e `hanmeimei`, mas ambos devolvem `hanmeimei`. O segundo pedido sobrescreve a variável estática `$name`; ao acordar o primeiro, o valor já é `hanmeimei`.

**Forma correta: armazenar o estado no context**

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

A classe `support\Context` armazena dados do contexto das coroutines. Quando a coroutine termina, os dados são eliminados automaticamente.
Com coroutines ativadas, cada pedido usa a sua própria coroutine, pelo que o context é limpo no final do pedido.
Sem coroutines, o context é limpo no final do pedido.

**Variáveis locais não poluem os dados**

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

Por ser `$name` uma variável local, as coroutines não podem aceder às variáveis locais umas das outras, pelo que o uso de variáveis locais é seguro neste ambiente.

## Locker

Se um componente ou lógica não foi pensado para coroutines, podem surgir conflitos de recursos ou problemas de atomicidade. Nestes casos usar `Workerman\Locker` para serializar o acesso:

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
        // Sem bloqueio, o Swoole pode lançar erros como "Socket#10 has already been bound to another coroutine#10"
        // O Swow pode provocar coredump
        // Com Fiber não há problema, a extensão Redis usa I/O bloqueante
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel (execução paralela)

Para executar várias tarefas em paralelo e recolher resultados, usar `Workerman\Parallel`:

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
        return json($results); // Resposta: [1,2,3,4]
    }

}
```

## Pool (agrupamento de ligações)

Várias coroutines a partilhar a mesma ligação podem corromper dados. Usar pools para bases de dados, Redis, etc.

O webman fornece [webman/database](../db/tutorial.md), [webman/redis](../db/redis.md), [webman/cache](../db/cache.md), [webman/think-orm](../db/thinkorm.md) e [webman/think-cache](../db/thinkcache.md). Todos incluem pools e funcionam com e sem coroutines.

Para adaptar um componente sem pool, usar `Workerman\Pool`. Exemplo:

**Componente de base de dados**

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

**Utilização**

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

## Mais sobre coroutines

Consultar a [documentação do Workerman sobre coroutines](https://www.workerman.net/doc/workerman/coroutine/coroutine.html).

## Deploy misto com e sem coroutines

O webman permite combinar serviços com e sem coroutines, por exemplo sem coroutines para tráfego normal e com coroutines para I/O lento, encaminhando o tráfego com nginx.

Exemplo em `config/process.php`:

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
    
    // ... resto da configuração omitido ...
];
```

Encaminhamento no nginx:

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
