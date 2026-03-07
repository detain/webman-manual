# Korutin

webman Workerman üzerine kuruludur, bu yüzden webman Workerman'ın korutin özelliklerini kullanabilir.
Korutinler üç sürücü destekler: `Swoole`, `Swow` ve `Fiber`.

## Ön koşullar

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- Swoole veya Swow eklentisi kurulu veya `composer require revolt/event-loop` (Fiber için)
- Korutinler varsayılan olarak kapalıdır; `eventLoop` ayarı ile ayrıca açılması gerekir

## Nasıl etkinleştirilir

webman süreç bazında farklı sürücüler destekler. `config/process.php` içinde (eklenti process.php yapılandırması dahil) `eventLoop` ile korutin sürücüsü ayarlanır:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // Varsayılan boş, Select veya Event otomatik seçilir, korutin kapalı
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
        // Korutin açmak için: Workerman\Events\Swoole::class, Workerman\Events\Swow::class veya Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... diğer yapılandırma atlanmış ...
];
```

> **İpucu**
> webman süreç bazında farklı `eventLoop` ayarlamaya izin verir, böylece belirli süreçlerde korutin açılabilir.
> Yukarıdaki örnekte 8787 portunda korutin kapalı, 8686 portunda açık. nginx ile yönlendirerek korutin ve korutinsiz karma dağıtım yapılabilir.

## Korutin örneği

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

`eventLoop` `Swoole`, `Swow` veya `Fiber` olduğunda webman her istek için bir korutin oluşturur ve işleyicide daha fazla korutin oluşturulabilir.

## Korutin sınırlamaları

* Swoole veya Swow kullanıldığında bloklayan I/O'da korutin otomatik geçiş yapar, senkron kod asenkron çalışır.
* Fiber sürücüsünde bloklayan I/O'da korutin geçişi olmaz, süreç bloklar.
* Korutin kullanırken aynı kaynağa (DB bağlantısı, dosya vb.) birden fazla korutinin aynı anda erişmesinden kaçının; bağlantı havuzu veya kilit kullanın.
* Korutin kullanırken istekle ilgili durumu global veya statik değişkende saklamayın; korutin bağlamı (`context`) kullanın.

## Diğer notlar

Swow PHP'nin bloklayan işlevlerini düşük seviyede kanca (hook) yapar, bu da PHP'nin bazı varsayılan davranışlarını değiştirir. Swow kurulu ama kullanılmıyorsa hata çıkabilir.

**Öneriler:**
* Proje Swow kullanmıyorsa Swow eklentisini kurmayın.
* Swow kullanıyorsa `eventLoop`'u `Workerman\Events\Swow::class` olarak ayarlayın.

## Korutin bağlamı

Korutin ortamında **istekle ilgili** durumu global veya statik değişkende saklamayın. Yanlış örnek:

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

> **Not**
> Global veya statik değişkenler tamamen yasak değil; yasak olan bunlarda **istekle ilgili durum** saklamaktır.
> Global yapılandırma, DB bağlantıları, singleton vb. global veya statik değişkende saklanabilir.

Süreç sayısı 1 iken art arda iki istek gönderildiğinde:
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

Beklenen `lilei` ve `hanmeimei` ama ikisi de `hanmeimei` döner. İkinci istek statik `$name`'i ezer, birincisi uyandığında değer zaten `hanmeimei` olur.

**Doğru yöntem: Durumu context'te saklayın**

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

`support\Context` sınıfı korutin bağlam verilerini saklar. Korutin bittiğinde ilgili bağlam verileri otomatik silinir.
Korutin ortamında her istek kendi korutininde çalışır, dolayısıyla istek bittiğinde context temizlenir.
Korutinsiz ortamda istek bittiğinde context temizlenir.

**Yerel değişkenler veri kirliliğine yol açmaz**

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

`$name` yerel değişken olduğundan korutinler birbirinin yerel değişkenlerine erişemez, dolayısıyla yerel değişken kullanımı korutin açısından güvenlidir.

## Locker

Bazı bileşen veya mantık korutin ortamını hesaba katmıyorsa kaynak çakışması veya atomiklik sorunları ortaya çıkabilir. Bu durumda erişimi serileştirmek için `Workerman\Locker` kullanın:

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
        // Kilitsiz: Swoole'da "Socket#10 has already been bound to another coroutine#10" gibi hata çıkabilir
        // Swow'da coredump olabilir
        // Fiber: Redis eklentisi senkron bloklayan I/O kullandığı için sorun yok
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel (paralel çalıştırma)

Birden fazla görevi paralel çalıştırıp sonuçları toplamak için `Workerman\Parallel` kullanın:

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
        return json($results); // Yanıt: [1,2,3,4]
    }

}
```

## Pool (bağlantı havuzu)

Birden fazla korutinin aynı bağlantıyı paylaşması veri bozulmasına yol açabilir. DB, Redis vb. için bağlantı havuzu kullanın.

webman zaten [webman/database](../db/tutorial.md), [webman/redis](../db/redis.md), [webman/cache](../db/cache.md), [webman/think-orm](../db/thinkorm.md), [webman/think-cache](../db/thinkcache.md) sağlıyor. Hepsi havuz içerir ve korutinli ve korutinsiz ortamlarda çalışır.

Havuzu olmayan bir bileşeni uyarlamak için `Workerman\Pool` kullanın. Örnek:

**Veritabanı bileşeni**

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

**Kullanım**

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

## Korutin ve ilgili bileşenler hakkında daha fazla

[Workerman korutin dokümantasyonuna](https://www.workerman.net/doc/workerman/coroutine/coroutine.html) bakın.

## Korutinli ve korutinsiz karma dağıtım

webman korutinli ve korutinsiz hizmetlerin birlikte çalışmasını destekler; örn. genel iş için korutinsiz, yavaş I/O için korutinli, nginx ile istekleri farklı hizmetlere yönlendirme.

Örnek `config/process.php`:

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
    
    // ... diğer yapılandırma atlanmış ...
];
```

nginx ile yönlendirme:

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
