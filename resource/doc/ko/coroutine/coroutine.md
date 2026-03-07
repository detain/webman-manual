# 코루틴

webman은 Workerman 기반으로 개발되어 webman에서 Workerman의 코루틴 기능을 사용할 수 있습니다.
코루틴은 `Swoole`, `Swow`, `Fiber` 세 가지 드라이버를 지원합니다.

## 사전 요구사항

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- Swoole 또는 Swow 확장 설치, 또는 `composer require revolt/event-loop`(Fiber용)
- 코루틴은 기본 비활성화되며, `eventLoop` 설정으로 별도 활성화해야 합니다

## 활성화 방법

webman은 프로세스별로 다른 드라이버를 설정할 수 있습니다. `config/process.php`(플러그인 process.php 설정 포함)에서 `eventLoop`로 코루틴 드라이버를 설정합니다:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // 기본값 빈 문자열, Select 또는 Event 자동 선택, 코루틴 비활성화
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
        // 코루틴 활성화: Workerman\Events\Swoole::class, Workerman\Events\Swow::class 또는 Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... 기타 설정 생략 ...
];
```

> **팁**
> webman은 프로세스별로 다른 `eventLoop`를 설정할 수 있어 특정 프로세스에만 코루틴을 활성화할 수 있습니다.
> 위 예시에서는 8787 포트는 코루틴 비활성화, 8686 포트는 코루틴 활성화입니다. nginx로 전달하면 코루틴/비코루틴 혼합 배포가 가능합니다.

## 코루틴 예제

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

`eventLoop`가 `Swoole`, `Swow`, `Fiber`일 때 webman은 요청별로 코루틴을 생성하며, 요청 처리 중 추가 코루틴을 생성할 수 있습니다.

## 코루틴 제한사항

* Swoole, Swow 사용 시 블로킹 I/O에서 코루틴이 자동 전환되어 동기 코드가 비동기로 실행됩니다.
* Fiber 드라이버는 블로킹 I/O에서 코루틴 전환이 일어나지 않고 프로세스가 블로킹됩니다.
* 코루틴 사용 시 DB 연결, 파일 작업 등 동일 리소스를 여러 코루틴이 동시에 사용하면 리소스 경합이 발생할 수 있습니다. 연결 풀이나 락을 사용하세요.
* 코루틴 사용 시 요청 관련 상태를 전역 변수나 정적 변수에 저장하지 마세요. 전역 데이터 오염이 발생할 수 있습니다. 코루틴 컨텍스트(`context`)를 사용하세요.

## 기타 참고사항

Swow는 PHP의 블로킹 함수를 후킹하는데, PHP의 일부 기본 동작에 영향을 주어 Swow를 사용하지 않는데 Swow가 설치되어 있으면 버그가 발생할 수 있습니다.

**권장 사항:**
* Swow를 사용하지 않는 프로젝트에서는 Swow 확장을 설치하지 마세요.
* Swow를 사용하는 프로젝트에서는 `eventLoop`를 `Workerman\Events\Swow::class`로 설정하세요.

## 코루틴 컨텍스트

코루틴 환경에서는 **요청 관련** 상태를 전역 변수나 정적 변수에 저장하지 마세요. 예:

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

> **참고**
> 전역 변수나 정적 변수 사용 자체가 금지된 것이 아니라, **요청 관련 상태 데이터**를 저장하는 것이 금지됩니다.
> 전역 설정, DB 연결, 싱글톤 등 전역 공유 객체는 전역/정적 변수에 저장해도 됩니다.

프로세스 수를 1로 하고 다음 두 요청을 연속 발송할 때:
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

각각 `lilei`, `hanmeimei`가 반환되길 기대하지만 실제로는 둘 다 `hanmeimei`가 반환됩니다. 두 번째 요청이 정적 변수 `$name`을 덮어써서 첫 번째 요청이 슬립에서 깨어날 때 이미 `hanmeimei`가 되기 때문입니다.

**올바른 방법: context에 요청 상태 저장**

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

`support\Context` 클래스는 코루틴 컨텍스트 데이터를 저장합니다. 코루틴 종료 시 해당 context 데이터는 자동 삭제됩니다.
코루틴 환경에서는 요청별로 별도 코루틴이 실행되므로 요청 완료 시 context가 자동으로 정리됩니다.
비코루틴 환경에서는 요청 종료 시 context가 정리됩니다.

**지역 변수는 데이터 오염을 일으키지 않음**

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

`$name`은 지역 변수라 코루틴 간에 서로의 지역 변수에 접근할 수 없어, 지역 변수 사용은 코루틴 안전합니다.

## Locker (잠금)

일부 컴포넌트나 로직이 코루틴 환경을 고려하지 않아 리소스 경합이나 원자성 문제가 발생할 수 있습니다. 이럴 때 `Workerman\Locker`로 잠금을 걸어 동시 접근을 직렬화할 수 있습니다:

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
        // 잠금 없이: Swoole에서 "Socket#10 has already been bound to another coroutine#10" 같은 오류 가능
        // Swow에서 coredump 가능
        // Fiber: Redis 확장이 동기 블로킹 I/O라 문제 없음
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel (병렬 실행)

여러 작업을 병렬로 실행하고 결과를 모으려면 `Workerman\Parallel`을 사용하세요:

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
        return json($results); // 응답: [1,2,3,4]
    }

}
```

## Pool (연결 풀)

여러 코루틴이 동일 연결을 공유하면 데이터가 꼬일 수 있습니다. DB, Redis 등은 연결 풀로 관리하세요.

webman은 이미 [webman/database](../db/tutorial.md), [webman/redis](../db/redis.md), [webman/cache](../db/cache.md), [webman/think-orm](../db/thinkorm.md), [webman/think-cache](../db/thinkcache.md)를 제공하며, 모두 연결 풀을 내장하고 코루틴/비코루틴 환경에서 사용할 수 있습니다.

연결 풀이 없는 컴포넌트를 적용하려면 `Workerman\Pool`을 사용할 수 있습니다. 예:

**DB 컴포넌트**

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

**사용**

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

## 코루틴 관련 추가 정보

[Workerman 코루틴 문서](https://www.workerman.net/doc/workerman/coroutine/coroutine.html)를 참조하세요.

## 코루틴/비코루틴 혼합 배포

webman은 코루틴과 비코루틴 서비스를 함께 운영할 수 있습니다. 예: 비코루틴으로 일반 업무, 코루틴으로 느린 I/O 업무 처리, nginx로 요청 전달.

예시 `config/process.php`:

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
    
    // ... 기타 설정 생략 ...
];
```

nginx로 요청 전달:

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
