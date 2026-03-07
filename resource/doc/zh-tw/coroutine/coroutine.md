# 協程

webman是基於workerman開發的，所以webman可以使用workerman的協程特性。
協程支援`Swoole` `Swow`和`Fiber`三種驅動。

## 前提條件

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- 安裝了swoole或者swow擴充功能，或者安裝了`composer require revolt/event-loop` (Fiber)
- 協程預設是關閉的，需要單獨設定eventLoop開啟

## 開啟方法

webman支援為不同的進程開啟不同的驅動，所以你可以在`config/process.php`(包括外掛process.php設定)中透過`eventLoop`設定協程驅動：

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // 預設為空自動選擇Select或者Event，不開啟協程
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
        // 開啟協程需要設定為 Workerman\Events\Swoole::class 或者 Workerman\Events\Swow::class 或者 Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... 其它設定省略 ...
];
```

> **提示**
> webman可以為不同的進程設定不同的eventLoop，這意味著你可以選擇性的為特定進程開啟協程。
> 例如上面設定中8787端口的服務沒有開啟協程，8686端口的服務開啟了協程，配合nginx轉發可以實現協程和非協程混合部署。

## 協程範例

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

當`eventLoop`為`Swoole` `Swow` `Fiber`時，webman會為每個請求建立一個協程來執行，在處理請求時可以繼續建立新的協程執行業務程式碼。

## 協程限制

* 當使用Swoole Swow為驅動時，業務遇到阻塞IO協程會自動切換，能實現同步程式碼非同步執行。
* 當使用Fiber驅動時，遇到阻塞IO時，協程不會發生切換，進程進入阻塞狀態。
* 使用協程時，不能多個協程同時對同一個資源進行操作，例如資料庫連線，檔案操作等，這可能會引起資源競爭，正確的用法是使用連線池或者鎖來保護資源。
* 使用協程時，不能將請求相關的狀態資料儲存在全域變數或者靜態變數中，這可能會引起全域資料污染，正確的用法是使用協程上下文的`context`來存取它們。

## 其它注意事項

Swow底層會自動hook php的阻塞函數，但是因為這種hook影響了PHP的某些預設行為，所以在你沒有使用Swow但卻裝了Swow時可能會產生bug。

**所以建議：**
* 如果你的專案沒有使用Swow時，請不要安裝Swow擴充功能
* 如果你的專案使用了Swow，請設定`eventLoop`為`Workerman\Events\Swow::class`

## 協程上下文

協程環境禁止將**請求相關**的狀態資訊儲存在全域變數或者靜態變數中，因為這可能會導致全域變數污染，例如

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

> **注意**
> 協程環境下並非禁止使用全域變數或靜態變數，而是禁止使用全域變數或靜態變數儲存**請求相關的狀態資料**。
> 例如全域設定、資料庫連線、一些類別的單例等需要全域共用的物件資料是推薦用全域變數或靜態變數儲存的。

將進程數設定為1，當我們連續發起兩個請求時
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei
我們期望兩個請求回傳的結果分別是 `lilei` 和 `hanmeimei`，但實際上回傳的都是`hanmeimei`。
這是因為第二個請求將靜態變數`$name`覆蓋了，第一個請求睡眠結束時回傳時靜態變數`$name`已經成為`hanmeimei`。

**正確的方法應該是使用context儲存請求狀態資料**

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

`support\Context`類別用於儲存協程上下文資料，當協程執行完畢後，相應的context資料會自動刪除。
協程環境裡，因為每個請求都是單獨的協程，所以當請求完成時context資料會自動銷毀。
非協程環境裡，context會在請求結束時會自動銷毀。

**區域變數不會造成資料污染**

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

因為`$name`是區域變數，協程之間無法互相存取區域變數，所以使用區域變數是協程安全的。

## Locker 鎖

有時候一些元件或者業務沒有考慮到協程環境，可能會出現資源競爭或原子性問題，這時候可以使用`Workerman\Locker`加鎖來實現排隊處理，防止並行問題。

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
        // 如果不加鎖，Swoole下會觸發類似 "Socket#10 has already been bound to another coroutine#10" 錯誤
        // Swow下可能會觸發coredump
        // Fiber下因為Redis擴充功能是同步阻塞IO，所以不會有問題
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel 並行執行

當我們需要並行執行多個任務並取得結果時，可以使用`Workerman\Parallel`來實現。

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
                // Do something
                return $i;
            });
        }
        $results = $parallel->wait();
        return json($results); // Response: [1,2,3,4]
    }

}
```

## Pool 連線池

多個協程共用同一個連線會導致資料混亂，所以需要使用連線池來管理資料庫、redis等連線資源。

webman已經提供了 [webman/database](../db/tutorial.md) [webman/redis](../db/redis.md) [webman/cache](../db/cache.md) [webman/think-orm](../db/thinkorm.md) [webman/think-cache](../db/thinkcache.md)等元件，它們都整合了連線池，支援在協程和非協程環境下使用。

如果你想改造一個沒有連線池的元件，可以使用`Workerman\Pool`來實現，參考如下程式碼。

**資料庫元件**

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
        // 從協程上下文中取得連線，保證同一個協程使用同一個連線
        $pdo = Context::get('pdo');
        if (!$pdo) {
            // 從連線池中取得連線
            $pdo = self::$pool->get();
            Context::set('pdo', $pdo);
            // 當協程結束時，自動歸還連線
            Coroutine::defer(function () use ($pdo) {
                self::$pool->put($pdo);
            });
        }
        return call_user_func_array([$pdo, $name], $arguments);
    }

    private static function initializePool(): void
    {
        // 建立一個連線池，最大連線數為10
        self::$pool = new Pool(10);
        // 設定連線建立器(為了簡潔，省略了設定檔讀取)
        self::$pool->setConnectionCreator(function () {
            return new \PDO('mysql:host=127.0.0.1;dbname=your_database', 'your_username', 'your_password');
        });
        // 設定連線關閉器
        self::$pool->setConnectionCloser(function ($pdo) {
            $pdo = null;
        });
        // 設定心跳偵測器
        self::$pool->setHeartbeatChecker(function ($pdo) {
            $pdo->query('SELECT 1');
        });
    }

}
```

**使用**

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

## 更多協程及相關元件介紹

參考[workerman 協程文件](https://www.workerman.net/doc/workerman/coroutine/coroutine.html)

## 協程與非協程混合部署

webman支援協程和非協程混合部署，例如非協程處理普通業務，協程處理慢IO業務，透過nginx轉發請求到不同的服務上。

例如 `config/process.php`

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // 預設為空自動選擇Select或者Event，不開啟協程
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
        // 開啟協程需要設定為 Workerman\Events\Swoole::class 或者 Workerman\Events\Swow::class 或者 Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ],
    
    // ... 其它設定省略 ...
];
```

然後透過nginx設定轉發請求到不同的服務上

```nginx
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# 新增一個8686 upstream
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # 以/tast開頭的請求走8686端口，請依實際情況將/tast更改為你需要的前綴
  location /tast {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # 其它請求走原8787端口
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
