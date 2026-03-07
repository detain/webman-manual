# করুটাইন

webman Workerman-এর উপর ভিত্তি করে তৈরি, তাই webman Workerman-এর করুটাইন বৈশিষ্ট্য ব্যবহার করতে পারে।
করুটাইন তিনটি ড্রাইভার সমর্থন করে: `Swoole`, `Swow` এবং `Fiber`।

## প্রাক-প্রয়োজনীয়তা

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- Swoole বা Swow এক্সটেনশন ইনস্টল করা, অথবা `composer require revolt/event-loop` (Fiber এর জন্য)
- করুটাইন ডিফল্টভাবে বন্ধ; `eventLoop` সেটিং দিয়ে আলাদা করে চালু করতে হবে

## সক্রিয় করার পদ্ধতি

webman প্রক্রিয়া অনুযায়ী ভিন্ন ড্রাইভার চালু করতে পারে। `config/process.php`-এ (প্লাগইন process.php কনফিগারেশন সহ) `eventLoop` দিয়ে করুটাইন ড্রাইভার সেট করা যায়:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // ডিফল্ট খালি, Select বা Event অটো নির্বাচন, করুটাইন বন্ধ
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
        // করুটাইন চালু করতে: Workerman\Events\Swoole::class অথবা Workerman\Events\Swow::class অথবা Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... অন্যগুলো বাদ ...
];
```

> **পরামর্শ**
> webman প্রক্রিয়া অনুযায়ী ভিন্ন `eventLoop` সেট করতে দেয়, তাই নির্দিষ্ট প্রক্রিয়ায় করুটাইন চালু করা যায়।
> উপরের উদাহরণে ৮৭৮৭ পোর্টে করুটাইন বন্ধ, ৮৬৮৬ পোর্টে চালু। nginx দিয়ে ফরওয়ার্ড করলে করুটাইন ও অ-করুটাইন মিশ্র ডিপ্লয় সম্ভব।

## করুটাইন উদাহরণ

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

`eventLoop` যখন `Swoole`, `Swow` বা `Fiber` তখন webman প্রতি অনুরোধে একটি করুটাইন তৈরি করে এবং হ্যান্ডলারে আরো করুটাইন তৈরি করা যায়।

## করুটাইন সীমাবদ্ধতা

* Swoole বা Swow ব্যবহারে ব্লকিং I/O-তে করুটাইন অটো সুইচ হয়, সিনক্রোনাস কোড অ্যাসিনক্রোনাসভাবে চলে।
* Fiber ড্রাইভারে ব্লকিং I/O-তে করুটাইন সুইচ হয় না, প্রক্রিয়া ব্লক হয়ে যায়।
* করুটাইন ব্যবহারে DB সংযোগ, ফাইল ইত্যাদি একই রিসোর্সে একসাথে কাজ করবেন না; কানেকশন পুল বা লক ব্যবহার করুন।
* করুটাইন ব্যবহারে অনুরোধ সংক্রান্ত স্টেট গ্লোবাল বা স্ট্যাটিক ভেরিয়েবলে রাখবেন না; করুটাইন কনটেক্সট (`context`) ব্যবহার করুন।

## অন্যান্য নোট

Swow PHP-এর ব্লকিং ফাংশন হুক করে, যা PHP-এর কিছু ডিফল্ট আচরণ বদলায়। Swow ইনস্টল থাকলে কিন্তু ব্যবহার না করলে বাগ হতে পারে।

**পরামর্শ:**
* Swow না ব্যবহার করলে Swow এক্সটেনশন ইনস্টল করবেন না।
* Swow ব্যবহার করলে `eventLoop` সেট করুন `Workerman\Events\Swow::class` এ।

## করুটাইন কনটেক্সট

করুটাইন এনভায়রনমেন্টে **অনুরোধ সংক্রান্ত** স্টেট গ্লোবাল বা স্ট্যাটিক ভেরিয়েবলে রাখবেন না। উদাহরণ:

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

> **নোট**
> গ্লোবাল বা স্ট্যাটিক ভেরিয়েবল নিষিদ্ধ নয়; নিষিদ্ধ শুধু **অনুরোধ সংক্রান্ত স্টেট** সেখানে রাখা।
> গ্লোবাল কনফিগ, DB সংযোগ, সিঙ্গলটন ইত্যাদি গ্লোবাল/স্ট্যাটিক ভেরিয়েবলে রাখা যায়।

প্রক্রিয়া সংখ্যা ১ রেখে দুটি অনুরোধ পাঠালে:
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

`lilei` ও `hanmeimei` প্রত্যাশা করা হয়, কিন্তু দুটোই `hanmeimei` ফেরায়। দ্বিতীয় অনুরোধ `$name` ওভাররাইট করে, প্রথমটার স্লিপ শেষে ইতিমধ্যে `hanmeimei` হয়ে যায়।

**সঠিক পদ্ধতি: context এ স্টেট রাখুন**

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

`support\Context` করুটাইন কনটেক্সট ডাটা জমা রাখে। করুটাইন শেষ হলে সংশ্লিষ্ট context ডাটা নিজে নিজে মুছে যায়।
করুটাইন এনভায়রনমেন্টে প্রতিটি অনুরোধ আলাদা করুটিনে চলে, তাই অনুরোধ শেষে context মুছে যায়।
অ-করুটাইন এনভায়রনমেন্টে অনুরোধ শেষে context মুছে যায়।

**লোকাল ভেরিয়েবল ডাটা দূষিত করে না**

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

`$name` লোকাল ভেরিয়েবল, করুটাইনগুলো একে অপরের লোকাল ভেরিয়েবল একসেস করতে পারে না, তাই লোকাল ভেরিয়েবল করুটাইন-সেফ।

## Locker

কিছু কম্পোনেন্ট বা লজিক করুটাইন এনভায়রনমেন্ট মাথায় রেখে তৈরি না হলে রিসোর্স কনফ্লিক্ট বা অ্যাটমিসিটি সমস্যা হতে পারে। সেক্ষেত্রে `Workerman\Locker` দিয়ে লক দিয়ে অ্যাক্সেস সিরিয়ালাইজ করুন:

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
        // লক ছাড়া: Swoole এ "Socket#10 has already been bound to another coroutine#10" মত এরর হতে পারে
        // Swow এ coredump হতে পারে
        // Fiber: Redis এক্সটেনশন সিনক্রোনাস ব্লকিং I/O, তাই সমস্যা নেই
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel (সমান্তরাল এক্সিকিউশন)

কয়েকটি টাস্ক সমান্তরালে চালাতে ও ফলাফল জোগাড় করতে `Workerman\Parallel` ব্যবহার করুন:

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
        return json($results); // রেসপন্স: [1,2,3,4]
    }

}
```

## Pool (কানেকশন পুল)

কয়েকটি করুটাইন একই কানেকশন শেয়ার করলে ডাটা গড়বড় হতে পারে। DB, Redis ইত্যাদির জন্য কানেকশন পুল ব্যবহার করুন।

webman ইতিমধ্যে [webman/database](../db/tutorial.md), [webman/redis](../db/redis.md), [webman/cache](../db/cache.md), [webman/think-orm](../db/thinkorm.md), [webman/think-cache](../db/thinkcache.md) দিচ্ছে। সবগুলিতে পুল আছে, করুটাইন ও অ-করুটাইন উভয় এনভায়রনমেন্টে চলে।

পুল ছাড়া কোনো কম্পোনেন্ট অ্যাডাপ্ট করতে `Workerman\Pool` ব্যবহার করুন। উদাহরণ:

**ডাটাবেস কম্পোনেন্ট**

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

**ব্যবহার**

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

## করুটাইন ও সংক্রান্ত কম্পোনেন্টের আরও তথ্য

[workerman করুটাইন ডকুমেন্টেশন](https://www.workerman.net/doc/workerman/coroutine/coroutine.html) দেখুন।

## করুটাইন ও অ-করুটাইন মিশ্র ডিপ্লয়

webman করুটাইন ও অ-করুটাইন উভয় ধরনের সেবা একসাথে চালানো সমর্থন করে; যেমন সাধারণ ব্যবসায়িক লজিক অ-করুটাইন, ধীর I/O করুটাইন, nginx দিয়ে রিকোয়েস্ট ফরওয়ার্ড করে ভিন্ন সার্ভিসে।

উদাহরণ `config/process.php`:

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
    
    // ... অন্যগুলো বাদ ...
];
```

nginx দিয়ে রিকোয়েস্ট ফরওয়ার্ড:

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
