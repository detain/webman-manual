# कॉरूटीन

webman Workerman पर आधारित है, इसलिए webman Workerman की कॉरूटीन सुविधाओं का उपयोग कर सकता है।
कॉरूटीन तीन ड्राइवर समर्थन करती है: `Swoole`, `Swow` और `Fiber`।

## पूर्वापेक्षाएँ

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- Swoole या Swow एक्सटेंशन इंस्टॉल, या `composer require revolt/event-loop` (Fiber के लिए)
- कॉरूटीन डिफ़ॉल्ट रूप से बंद है; `eventLoop` सेटिंग से अलग से चालू करना होगा

## सक्षम करने की विधि

webman प्रक्रिया के अनुसार अलग ड्राइवर सक्षम कर सकता है। `config/process.php` में (प्लगइन process.php कॉन्फ़िगरेशन सहित) `eventLoop` से कॉरूटीन ड्राइवर सेट करें:

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // डिफ़ॉल्ट खाली, Select या Event स्वचालित चुनाव, कॉरूटीन बंद
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
        // कॉरूटीन चालू: Workerman\Events\Swoole::class या Workerman\Events\Swow::class या Workerman\Events\Fiber::class
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... बाकी ओमिट ...
];
```

> **सुझाव**
> webman प्रक्रिया के अनुसार अलग `eventLoop` सेट करने देता है, इसलिए विशिष्ट प्रक्रिया में कॉरूटीन चालू की जा सकती है।
> ऊपर के उदाहरण में पोर्ट 8787 पर कॉरूटीन बंद, 8686 पर चालू। nginx से फॉरवर्ड करके कॉरूटीन और नॉन-कॉरूटीन मिश्रित तैनाती संभव है।

## कॉरूटीन उदाहरण

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

`eventLoop` जब `Swoole`, `Swow` या `Fiber` हो तो webman प्रत्येक अनुरोध के लिए एक कॉरूटीन बनाता है और हेंडलर में और बना सकता है।

## कॉरूटीन सीमाएँ

* Swoole या Swow पर ब्लॉकिंग I/O पर कॉरूटीन स्वतः स्विच होती है, सिंक्रोनस कोड एसिंक्रोनस चलता है।
* Fiber ड्राइवर पर ब्लॉकिंग I/O पर कॉरूटीन स्विच नहीं होती, प्रक्रिया ब्लॉक होती है।
* कॉरूटीन उपयोग में DB कनेक्शन, फाइल आदि एक ही रिसोर्स पर कई कॉरूटीन एकसाथ न करें; कनेक्शन पूल या लॉक इस्तेमाल करें।
* कॉरूटीन उपयोग में अनुरोध संबंधी स्टेट ग्लोबल या स्टेटिक वेरिएबल में न रखें; कॉरूटीन कॉन्टेक्सट (`context`) इस्तेमाल करें।

## अन्य नोट्स

Swow PHP की ब्लॉकिंग फंक्शन को हुक करता है, जो PHP के कुछ डिफ़ॉल्ट व्यवहार बदलता है। Swow इंस्टॉल हो पर उपयोग न हो तो बग हो सकता है।

**सिफारिशें:**
* Swow इस्तेमाल न करने वाले प्रोजेक्ट में Swow एक्सटेंशन इंस्टॉल न करें।
* Swow इस्तेमाल करने वाले प्रोजेक्ट में `eventLoop` सेट करें `Workerman\Events\Swow::class` पर।

## कॉरूटीन कॉन्टेक्सट

कॉरूटीन वातावरण में **अनुरोध संबंधी** स्टेट ग्लोबल या स्टेटिक वेरिएबल में न रखें। उदाहरण गलत तरीका:

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

> **नोट**
> ग्लोबल या स्टेटिक वेरिएबल पूरी तरह वर्जित नहीं; वर्जित है **अनुरोध संबंधी स्टेट** वहाँ रखना।
> ग्लोबल कॉन्फ़िग, DB कनेक्शन, सिंगलटन आदि ग्लोबल/स्टेटिक वेरिएबल में रखे जा सकते हैं।

प्रक्रिया संख्या 1 रखकर दो अनुरोध लगातार भेजने पर:
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

`lilei` और `hanmeimei` अपेक्षित, लेकिन दोनों `hanmeimei` लौटाते हैं। दूसरा अनुरोध `$name` ओवरराइट करता है, पहले का स्लीप खत्म होने तक पहले से `hanmeimei` हो चुका होता है।

**सही तरीका: context में स्टेट रखें**

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

`support\Context` कॉरूटीन कॉन्टेक्सट डेटा जमा करता है। कॉरूटीन खत्म होने पर संबंधित कॉन्टेक्सट डेटा स्वतः हट जाता है।
कॉरूटीन वातावरण में प्रत्येक अनुरोध अलग कॉरूटीन में चलता है, इसलिए अनुरोध खत्म होने पर कॉन्टेक्सट साफ हो जाता है।
नॉन-कॉरूटीन वातावरण में अनुरोध खत्म होने पर कॉन्टेक्सट साफ होता है।

**लोकल वेरिएबल डेटा दूषित नहीं करते**

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

`$name` लोकल वेरिएबल है, कॉरूटीन एक-दूसरे के लोकल वेरिएबल एक्सेस नहीं कर सकतीं, इसलिए लोकल वेरिएबल कॉरूटीन-सुरक्षित है।

## Locker

कुछ कंपोनेंट या बिजनेस लॉजिक कॉरूटीन वातावरण ध्यान में न रखता हो तो रिसोर्स कंफ्लिक्ट या एटॉमिसिटी समस्या हो सकती है। ऐसे में `Workerman\Locker` से लॉक कर एक्सेस सीरियलाइज़ करें:

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
        // बिना लॉक: Swoole में "Socket#10 has already been bound to another coroutine#10" जैसा एरर हो सकता है
        // Swow में coredump हो सकता है
        // Fiber: Redis एक्सटेंशन सिंक्रोनस ब्लॉकिंग I/O, कोई समस्या नहीं
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel (समानांतर निष्पादन)

कई टास्क समानांतर चलाने और नतीजे इकट्ठा करने के लिए `Workerman\Parallel` इस्तेमाल करें:

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

## Pool (कनेक्शन पूल)

कई कॉरूटीन एक ही कनेक्शन शेयर करें तो डेटा मिश्रित हो सकता है। DB, Redis आदि के लिए कनेक्शन पूल इस्तेमाल करें।

webman में पहले से [webman/database](../db/tutorial.md), [webman/redis](../db/redis.md), [webman/cache](../db/cache.md), [webman/think-orm](../db/thinkorm.md), [webman/think-cache](../db/thinkcache.md) हैं। सभी में पूल है, कॉरूटीन व नॉन-कॉरूटीन दोनों वातावरण में चलते हैं।

पूल रहित कंपोनेंट एडॉप्ट करने के लिए `Workerman\Pool` इस्तेमाल करें। उदाहरण:

**डेटाबेस कंपोनेंट**

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

**उपयोग**

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

## कॉरूटीन और संबंधित कंपोनेंट अधिक जानकारी

[workerman कॉरूटीन दस्तावेज़](https://www.workerman.net/doc/workerman/coroutine/coroutine.html) देखें।

## कॉरूटीन और नॉन-कॉरूटीन मिश्रित तैनाती

webman कॉरूटीन और नॉन-कॉरूटीन दोनों सर्विस एकसाथ चलाने का समर्थन करता है; जैसे सामान्य बिजनेस नॉन-कॉरूटीन, धीमा I/O वाला कॉरूटीन, nginx से अनुरोध अलग सर्विस पर फॉरवर्ड।

उदाहरण `config/process.php`:

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
    
    // ... बाकी ओमिट ...
];
```

nginx से अनुरोध फॉरवर्ड:

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
