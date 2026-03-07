# Coroutine

webman สร้างขึ้นบน Workerman ดังนั้น webman สามารถใช้ฟีเจอร์ coroutine ของ Workerman ได้
Coroutine รองรับสามไดรเวอร์: `Swoole`, `Swow` และ `Fiber`

## ข้อกำหนดเบื้องต้น

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- ติดตั้ง extension Swoole หรือ Swow หรือ `composer require revolt/event-loop` (สำหรับ Fiber)
- Coroutine ปิดโดยค่าเริ่มต้น ต้องเปิดผ่านการตั้งค่า `eventLoop`

## วิธีเปิดใช้งาน

webman รองรับการตั้งค่าไดรเวอร์ coroutine ที่แตกต่างกันสำหรับแต่ละ process ใน `config/process.php` (รวมถึงการตั้งค่า process.php ของปลั๊กอิน) ตั้งค่าผ่าน `eventLoop`:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // ค่าเริ่มต้นว่าง เลือก Select หรือ Event อัตโนมัติ ไม่เปิด coroutine
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
        // เปิด coroutine: ตั้งเป็น Workerman\Events\Swoole::class หรือ Workerman\Events\Swow::class หรือ Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... การตั้งค่าอื่นๆ ละไว้ ...
];
```

> **เคล็ดลับ**
> webman อนุญาตให้ตั้ง `eventLoop` ที่แตกต่างกันสำหรับแต่ละ process จึงสามารถเปิด coroutine เฉพาะ process ที่ต้องการได้
> ในตัวอย่างด้านบน พอร์ต 8787 ไม่เปิด coroutine พอร์ต 8686 เปิด coroutine ร่วมกับ nginx จะสามารถ deploy แบบผสม coroutine และ non-coroutine ได้

## ตัวอย่าง Coroutine

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

เมื่อ `eventLoop` เป็น `Swoole`, `Swow` หรือ `Fiber` webman จะสร้าง coroutine สำหรับแต่ละ request และสามารถสร้าง coroutine เพิ่มใน handler ได้

## ข้อจำกัดของ Coroutine

* เมื่อใช้ Swoole หรือ Swow เมื่อเจอ blocking I/O coroutine จะสลับโดยอัตโนมัติ โค้ดแบบ synchronous จะทำงานแบบ asynchronous
* เมื่อใช้ Fiber blocking I/O จะไม่ทำให้ coroutine สลับ process จะ block แทน
* เมื่อใช้ coroutine ห้ามให้ coroutine หลายตัวทำงานกับ resource เดียวกัน (เช่น connection DB, file) โดยไม่มีการป้องกัน ให้ใช้ connection pool หรือ lock
* เมื่อใช้ coroutine ห้ามเก็บ state ที่เกี่ยวกับ request ใน global หรือ static variable ให้ใช้ coroutine context (`context`) แทน

## หมายเหตุอื่นๆ

Swow จะ hook ฟังก์ชัน blocking ของ PHP ที่ระดับต่ำ ซึ่งเปลี่ยนพฤติกรรมบางอย่างของ PHP หากติดตั้ง Swow แต่ไม่ได้ใช้ อาจทำให้เกิด bug

**คำแนะนำ:**
* หากโปรเจกต์ไม่ได้ใช้ Swow ไม่ต้องติดตั้ง extension Swow
* หากโปรเจกต์ใช้ Swow ให้ตั้ง `eventLoop` เป็น `Workerman\Events\Swow::class`

## Coroutine Context

ในสภาพแวดล้อม coroutine ห้ามเก็บ **state ที่เกี่ยวกับ request** ใน global หรือ static variable ตัวอย่างผิด:

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

> **หมายเหตุ**
> การใช้ global หรือ static variable ไม่ได้ห้ามทั้งหมด แต่ห้ามเก็บ **state ที่เกี่ยวกับ request** ไว้ในนั้น
> การตั้งค่าแบบ global, connection DB, singleton ฯลฯ สามารถเก็บใน global หรือ static variable ได้

เมื่อตั้งจำนวน process เป็น 1 และส่ง request สองครั้งติดกัน:
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

คาดว่าจะได้ `lilei` และ `hanmeimei` แต่ทั้งคู่คืนค่า `hanmeimei` เพราะ request ที่สองเขียนทับ static variable `$name` เมื่อ request แรกตื่นจาก sleep ค่าใน static variable เป็น `hanmeimei` แล้ว

**วิธีที่ถูกต้อง: เก็บ state ใน context**

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

คลาส `support\Context` ใช้เก็บข้อมูล coroutine context เมื่อ coroutine จบ ข้อมูล context ที่เกี่ยวข้องจะถูกลบอัตโนมัติ
ในสภาพแวดล้อม coroutine แต่ละ request มี coroutine ของตัวเอง ดังนั้น context จะถูกล้างเมื่อ request เสร็จ
ในสภาพแวดล้อมที่ไม่ใช่ coroutine context จะถูกล้างเมื่อ request จบ

**ตัวแปร local ไม่ทำให้ข้อมูลปนกัน**

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

เนื่องจาก `$name` เป็นตัวแปร local coroutine ไม่สามารถเข้าถึงตัวแปร local ของกันและกันได้ การใช้ตัวแปร local จึงปลอดภัยในสภาพแวดล้อม coroutine

## Locker

เมื่อมี component หรือ business บางตัวไม่ได้คำนึงถึง coroutine อาจเกิด resource contention หรือปัญหาความ atomicity ในกรณีนี้ใช้ `Workerman\Locker` เพื่อล็อกและจัดลำดับการเข้าถึง:

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
        // ไม่มี lock: Swoole อาจเกิด error แบบ "Socket#10 has already been bound to another coroutine#10"
        // Swow อาจเกิด coredump
        // Fiber: ไม่มีปัญหาเพราะ extension Redis ใช้ blocking I/O แบบ sync
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel (การทำงานพร้อมกัน)

เมื่อต้องรันหลายงานพร้อมกันและรวบรวมผล ใช้ `Workerman\Parallel`:

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
        return json($results); // Response: [1,2,3,4]
    }

}
```

## Pool (Connection Pool)

coroutine หลายตัวใช้ connection เดียวกันจะทำให้ข้อมูลปนกัน ให้ใช้ connection pool สำหรับ DB, Redis เป็นต้น

webman มี [webman/database](../db/tutorial.md), [webman/redis](../db/redis.md), [webman/cache](../db/cache.md), [webman/think-orm](../db/thinkorm.md), [webman/think-cache](../db/thinkcache.md) อยู่แล้ว ทั้งหมดมี connection pool และใช้งานได้ทั้งในสภาพแวดล้อม coroutine และ non-coroutine

หากต้องการปรับ component ที่ไม่มี pool ให้ใช้ `Workerman\Pool` ได้ ตัวอย่าง:

**Database component**

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

**การใช้งาน**

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

## ข้อมูลเพิ่มเติมเกี่ยวกับ Coroutine

ดู [เอกสาร Workerman Coroutine](https://www.workerman.net/doc/workerman/coroutine/coroutine.html)

## การ Deploy แบบผสม Coroutine และ Non-Coroutine

webman รองรับการ deploy แบบผสม coroutine และ non-coroutine เช่น non-coroutine สำหรับ business ทั่วไป coroutine สำหรับ business ที่มี I/O ช้า ส่ง request ผ่าน nginx ไปยัง service ที่แตกต่างกัน

ตัวอย่าง `config/process.php`:

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
    
    // ... การตั้งค่าอื่นๆ ละไว้ ...
];
```

ส่ง request ผ่าน nginx:

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
