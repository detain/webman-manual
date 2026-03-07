# الكوروتينات (Coroutines)

webman مبني على Workerman، لذلك يمكن لـ webman استخدام ميزات كوروتينات Workerman.
تدعم الكوروتينات ثلاثة برامج تشغيل: `Swoole` و `Swow` و `Fiber`.

## المتطلبات المسبقة

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- تثبيت امتداد Swoole أو Swow، أو `composer require revolt/event-loop` (لـ Fiber)
- الكوروتينات معطلة افتراضياً ويجب تفعيلها عبر إعداد `eventLoop`

## طريقة التفعيل

يدعم webman برامج تشغيل كوروتينات مختلفة لكل عملية. في `config/process.php` (بما في ذلك إعداد process.php للمكوّنات الإضافية)، يمكن التكوين عبر `eventLoop`:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // فارغ افتراضياً، اختيار تلقائي Select أو Event، الكوروتينات معطلة
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
        // لتفعيل الكوروتينات: Workerman\Events\Swoole::class أو Workerman\Events\Swow::class أو Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... باقي التكوين محذوف ...
];
```

> **تلميح**
> يسمح webman بإعداد `eventLoop` مختلف لكل عملية، لذا يمكنك تفعيل الكوروتينات لعملات محددة فقط.
> في المثال أعلاه، المنفذ 8787 معطل الكوروتينات والمنفذ 8686 مفعل. مع توجيه nginx يمكن نشر مزيج من الخدمات مع وبدون كوروتينات.

## مثال على الكوروتين

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

عندما يكون `eventLoop` هو `Swoole` أو `Swow` أو `Fiber`، ينشئ webman كوروتيناً لكل طلب ويسمح بإنشاء المزيد داخل معالج الطلب.

## قيود الكوروتينات

* مع Swoole أو Swow، عند مواجهة I/O حاجز يتم تبديل الكوروتين تلقائياً، ويُنفذ الكود المتزامن بشكل غير متزامن.
* مع Fiber، لا يحدث تبديل عند I/O الحاجز بل تتعطل العملية.
* تجنب استخدام عدة كوروتينات للمورد نفسه (اتصال قاعدة البيانات، ملف، إلخ) بدون حماية. استخدم تجمعات الاتصالات أو القفل.
* لا تخزن الحالة المرتبطة بالطلب في متغيرات عامة أو ثابتة. استخدم سياق الكوروتين (`context`).

## ملاحظات أخرى

يقوم Swow بربط دوال PHP الحاجزة على مستوى منخفض، مما يغيّر جزءاً من سلوك PHP الافتراضي. إذا كان Swow مثبتاً لكن غير مستخدم، قد يسبب أخطاء.

**التوصيات:**
* إذا لم يكن المشروع يستخدم Swow: لا تثبت امتداد Swow.
* إذا كان يستخدم Swow: عيّن `eventLoop` إلى `Workerman\Events\Swow::class`.

## سياق الكوروتين

في بيئة الكوروتينات، لا تخزن **الحالة المرتبطة بالطلب** في متغيرات عامة أو ثابتة. مثال خاطئ:

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

> **ملاحظة**
> المتغيرات العامة أو الثابتة ليست ممنوعة بالكامل؛ الممنوع هو تخزين **الحالة المرتبطة بالطلب** فيها.
> التكوين العام واتصالات قاعدة البيانات والـ singletons يمكن تخزينها في متغيرات عامة أو ثابتة.

مع عملية واحدة وطلبان متتابعان:
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

المتوقع `lilei` و `hanmeimei`، لكن كلا الطلبين يعيد `hanmeimei`. الطلب الثاني يستبدل المتغير الثابت `$name` وعند استيقاظ الأول تكون القيمة بالفعل `hanmeimei`.

**الطريقة الصحيحة: تخزين الحالة في context**

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

تخزن فئة `support\Context` بيانات سياق الكوروتين. عند انتهاء الكوروتين تُحذف البيانات تلقائياً.
في بيئة الكوروتينات، كل طلب له كوروتين خاص به، لذا يُنظّف السياق عند انتهاء الطلب.
بدون كوروتينات، يُنظّف السياق عند انتهاء الطلب.

**المتغيرات المحلية لا تسبب تلوث البيانات**

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

بما أن `$name` متغير محلي، لا تستطيع الكوروتينات الوصول لمتغيرات بعضها المحلية، لذا استخدام المتغيرات المحلية آمن في بيئة الكوروتينات.

## Locker (القفل)

عندما لا يكون مكوّن أو منطق مصمماً للكوروتينات، قد تحدث منافسة على الموارد أو مشكلات ذرية. في هذه الحالات استخدم `Workerman\Locker` لتسلسل الوصول:

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
        // بدون قفل: قد يرمي Swoole أخطاء مثل "Socket#10 has already been bound to another coroutine#10"
        // قد يتسبب Swow في coredump
        // Fiber: لا مشكلة لأن امتداد Redis يستخدم I/O حاجز
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel (التنفيذ المتوازي)

لتشغيل مهام متعددة بالتوازي وجمع النتائج، استخدم `Workerman\Parallel`:

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
        return json($results); // الاستجابة: [1,2,3,4]
    }

}
```

## Pool (تجمع الاتصالات)

استخدام عدة كوروتينات لنفس الاتصال قد يفسد البيانات. استخدم تجمع اتصالات لقاعدة البيانات وRedis وغيرها.

يوفر webman بالفعل [webman/database](../db/tutorial.md) و [webman/redis](../db/redis.md) و [webman/cache](../db/cache.md) و [webman/think-orm](../db/thinkorm.md) و [webman/think-cache](../db/thinkcache.md). كلها تتضمن تجمعات وصول وتعمل في بيئات مع وبدون كوروتينات.

لتكيّف مكوّن بدون تجمع، استخدم `Workerman\Pool`. مثال:

**مكوّن قاعدة البيانات**

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

**الاستخدام**

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

## المزيد عن الكوروتينات

راجع [وثائق Workerman للكوروتينات](https://www.workerman.net/doc/workerman/coroutine/coroutine.html).

## نشر مختلط مع وبدون كوروتينات

يدعم webman تشغيل خدمات مع وبدون كوروتينات معاً، مثلاً بدون كوروتينات للمعالجة العادية ومع كوروتينات للـ I/O البطيء، مع توجيه الطلبات عبر nginx.

مثال `config/process.php`:

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
    
    // ... باقي التكوين محذوف ...
];
```

توجيه nginx:

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
