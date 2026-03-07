# Coroutine

webman được xây dựng trên Workerman, nên webman có thể sử dụng tính năng coroutine của Workerman.
Coroutine hỗ trợ ba driver: `Swoole`, `Swow` và `Fiber`.

## Điều kiện tiên quyết

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- Cài extension Swoole hoặc Swow, hoặc `composer require revolt/event-loop` (cho Fiber)
- Coroutine mặc định tắt; cần bật riêng qua cấu hình `eventLoop`

## Cách bật

webman hỗ trợ driver coroutine khác nhau cho từng process. Trong `config/process.php` (gồm cấu hình process.php của plugin), cấu hình qua `eventLoop`:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // Mặc định trống, tự chọn Select hoặc Event, coroutine tắt
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
        // Bật coroutine: Workerman\Events\Swoole::class, Workerman\Events\Swow::class hoặc Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... cấu hình khác lược bỏ ...
];
```

> **Mẹo**
> webman cho phép cấu hình `eventLoop` khác nhau theo process, nên bạn có thể bật coroutine chỉ cho process cụ thể.
> Trong ví dụ trên, cổng 8787 tắt coroutine, cổng 8686 bật. Kết hợp nginx chuyển tiếp có thể triển khai hỗn hợp coroutine và non-coroutine.

## Ví dụ coroutine

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

Khi `eventLoop` là `Swoole`, `Swow` hoặc `Fiber`, webman tạo một coroutine cho mỗi request và có thể tạo thêm trong handler.

## Hạn chế coroutine

* Khi dùng Swoole hoặc Swow, gặp I/O chặn thì coroutine tự chuyển, mã đồng bộ chạy bất đồng bộ.
* Khi dùng Fiber, I/O chặn không gây chuyển coroutine; process bị chặn.
* Khi dùng coroutine không để nhiều coroutine cùng truy cập một tài nguyên (kết nối DB, file...) mà không bảo vệ. Dùng pool kết nối hoặc lock.
* Khi dùng coroutine không lưu trạng thái liên quan request vào biến toàn cục hoặc tĩnh. Dùng ngữ cảnh coroutine (`context`).

## Ghi chú khác

Swow hook các hàm chặn của PHP ở tầng thấp, làm thay đổi một phần hành vi mặc định của PHP. Nếu cài Swow nhưng không dùng có thể gây lỗi.

**Khuyến nghị:**
* Nếu dự án không dùng Swow: không cài extension Swow.
* Nếu dùng Swow: cấu hình `eventLoop` thành `Workerman\Events\Swow::class`.

## Ngữ cảnh coroutine

Trong môi trường coroutine không lưu **trạng thái liên quan request** vào biến toàn cục hoặc tĩnh. Ví dụ sai:

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

> **Lưu ý**
> Biến toàn cục hoặc tĩnh không bị cấm hoàn toàn; cấm là lưu **trạng thái liên quan request** trong đó.
> Cấu hình toàn cục, kết nối DB, singleton... có thể lưu vào biến toàn cục/tĩnh.

Khi đặt số process là 1 và gửi hai request liên tiếp:
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

Kỳ vọng `lilei` và `hanmeimei`, nhưng cả hai trả về `hanmeimei`. Request thứ hai ghi đè biến tĩnh `$name`; khi request thứ nhất thức dậy giá trị đã là `hanmeimei`.

**Cách đúng: lưu trạng thái trong context**

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

Lớp `support\Context` lưu dữ liệu ngữ cảnh coroutine. Khi coroutine kết thúc, dữ liệu context tương ứng tự xóa.
Trong môi trường coroutine, mỗi request chạy trong coroutine riêng nên context tự hủy khi request hoàn thành.
Trong môi trường không coroutine, context hủy khi request kết thúc.

**Biến cục bộ không gây ô nhiễm dữ liệu**

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

Vì `$name` là biến cục bộ, các coroutine không truy cập biến cục bộ của nhau nên dùng biến cục bộ an toàn trong môi trường coroutine.

## Locker

Khi một số component hoặc logic chưa tính đến môi trường coroutine, có thể xảy ra tranh chấp tài nguyên hoặc vấn đề atomicity. Lúc đó dùng `Workerman\Locker` để khóa và tuần tự hóa truy cập:

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
        // Không khóa: Swoole có thể báo lỗi kiểu "Socket#10 has already been bound to another coroutine#10"
        // Swow có thể gây coredump
        // Fiber: không vấn đề vì extension Redis dùng I/O chặn đồng bộ
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel (chạy song song)

Khi cần chạy nhiều tác vụ song song và thu kết quả, dùng `Workerman\Parallel`:

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
        return json($results); // Phản hồi: [1,2,3,4]
    }

}
```

## Pool (pool kết nối)

Nhiều coroutine dùng chung một kết nối có thể làm hỏng dữ liệu. Dùng pool cho DB, Redis, v.v.

webman đã cung cấp [webman/database](../db/tutorial.md), [webman/redis](../db/redis.md), [webman/cache](../db/cache.md), [webman/think-orm](../db/thinkorm.md), [webman/think-cache](../db/thinkcache.md). Tất cả có pool và chạy được cả môi trường coroutine và non-coroutine.

Để điều chỉnh component không có pool, dùng `Workerman\Pool`. Ví dụ:

**Component cơ sở dữ liệu**

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

**Sử dụng**

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

## Thông tin thêm về coroutine

Tham khảo [tài liệu Workerman Coroutine](https://www.workerman.net/doc/workerman/coroutine/coroutine.html).

## Triển khai hỗn hợp coroutine và non-coroutine

webman hỗ trợ chạy đồng thời dịch vụ coroutine và non-coroutine, ví dụ non-coroutine cho nghiệp vụ thường, coroutine cho I/O chậm, chuyển request qua nginx đến dịch vụ khác nhau.

Ví dụ `config/process.php`:

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
    
    // ... cấu hình khác lược bỏ ...
];
```

Chuyển request qua nginx:

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
