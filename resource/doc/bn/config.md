# কনফিগারেশন ফাইল

## অবস্থান
webman কনফিগারেশন ফাইলগুলি `config/` ডিরেক্টরিতে থাকে। আপনি আপনার প্রকল্পে সংশ্লিষ্ট কনফিগারেশনে অ্যাক্সেস করতে `config()` ফাংশন ব্যবহার করতে পারেন।

## কনফিগারেশনে অ্যাক্সেস

সমস্ত কনফিগারেশন পেতে:
```php
config();
```

`config/app.php`-এ সমস্ত কনফিগারেশন পেতে:
```php
config('app');
```

`config/app.php`-এ `debug` কনফিগারেশন পেতে:
```php
config('app.debug');
```

কনফিগারেশন যদি একটি অ্যারে হয়, তাহলে নেস্টেড মান অ্যাক্সেস করতে আপনি `.` ব্যবহার করতে পারেন। উদাহরণ:
```php
config('file.key1.key2');
```

## ডিফল্ট মান
```php
config($key, $default);
```
ডিফল্ট মান দ্বিতীয় প্যারামিটার হিসেবে পাঠান। কনফিগারেশন না থাকলে ডিফল্ট মান ফেরত দেওয়া হবে। কনফিগারেশন না থাকলে এবং কোনো ডিফল্ট সেট না থাকলে `null` ফেরত দেওয়া হবে।

## কাস্টম কনফিগারেশন
ডেভেলপাররা `config/` ডিরেক্টরিতে নিজস্ব কনফিগারেশন ফাইল যোগ করতে পারেন। উদাহরণ:

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**কনফিগারেশনে অ্যাক্সেস করার সময় ব্যবহার**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## কনফিগারেশন পরিবর্তন করা
Webman ডায়নামিক কনফিগারেশন পরিবর্তন সমর্থন করে না। সমস্ত কনফিগারেশন সংশ্লিষ্ট কনফিগারেশন ফাইলে ম্যানুয়ালি পরিবর্তন করতে হবে, তারপর অ্যাপ্লিকেশন রিলোড বা পুনরায় চালু করতে হবে।

> **নোট**
> সার্ভার কনফিগারেশন `config/server.php` এবং প্রসেস কনফিগারেশন `config/process.php` রিলোড সমর্থন করে না। পরিবর্তন কার্যকর করতে আপনাকে পুনরায় চালু করতে হবে।

## গুরুত্বপূর্ণ অনুস্মারক
আপনি যদি `config/` এর অধীনে কোনো সাবডিরেক্টরিতে কনফিগারেশন ফাইল তৈরি করেন, উদাহরণস্বরূপ `config/order/status.php`, তাহলে আপনাকে `config/order/` ডিরেক্টরিতে নিম্নোক্ত বিষয়বস্তু সহ একটি `app.php` ফাইল প্রয়োজন হবে:
```php
<?php
return [
    'enable' => true,
];
```
`enable` কে `true` এ সেট করলে ফ্রেমওয়ার্ককে এই ডিরেক্টরি থেকে কনফিগারেশন লোড করতে বলা হয়।
কনফিগারেশন ডিরেক্টরি কাঠামো এরকম হতে হবে:
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
এরপর আপনি `status.php` থেকে অ্যারে বা নির্দিষ্ট কী `config.order.status` এর মাধ্যমে অ্যাক্সেস করতে পারবেন।


## কনফিগারেশন ফাইল রেফারেন্স

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // শোনার পোর্ট (1.6.0 থেকে সরানো হয়েছে, config/process.php এ কনফিগার করা)
    'transport' => 'tcp', // ট্রান্সপোর্ট প্রোটোকল (1.6.0 থেকে সরানো হয়েছে, config/process.php এ কনফিগার করা)
    'context' => [], // SSL ইত্যাদি (1.6.0 থেকে সরানো হয়েছে, config/process.php এ কনফিগার করা)
    'name' => 'webman', // প্রসেস নাম (1.6.0 থেকে সরানো হয়েছে, config/process.php এ কনফিগার করা)
    'count' => cpu_count() * 4, // প্রসেস সংখ্যা (1.6.0 থেকে সরানো হয়েছে, config/process.php এ কনফিগার করা)
    'user' => '', // ব্যবহারকারী (1.6.0 থেকে সরানো হয়েছে, config/process.php এ কনফিগার করা)
    'group' => '', // গ্রুপ (1.6.0 থেকে সরানো হয়েছে, config/process.php এ কনফিগার করা)
    'reusePort' => false, // পোর্ট পুনঃব্যবহার সক্ষম করুন (1.6.0 থেকে সরানো হয়েছে, config/process.php এ কনফিগার করা)
    'event_loop' => '',  // ইভেন্ট লুপ ক্লাস, ডিফল্টে স্বয়ংক্রিয়ভাবে নির্বাচিত
    'stop_timeout' => 2, // স্টপ/রিস্টার্ট/রিলোড সিগন্যাল পাওয়ার সময় সর্বোচ্চ অপেক্ষার সময়, সময়মতো প্রসেস বের না হলে জোর করে প্রস্থান
    'pid_file' => runtime_path() . '/webman.pid', // PID ফাইল অবস্থান
    'status_file' => runtime_path() . '/webman.status', // স্ট্যাটাস ফাইল অবস্থান
    'stdout_file' => runtime_path() . '/logs/stdout.log', // stdout ফাইল অবস্থান, webman চালু হওয়ার পর সকল আউটপুট এখানে লেখা হয়
    'log_file' => runtime_path() . '/logs/workerman.log', // Workerman লগ ফাইল অবস্থান
    'max_package_size' => 10 * 1024 * 1024 // সর্বোচ্চ প্যাকেট আকার, 10M। আপলোড ফাইল আকার এ দ্বারা সীমাবদ্ধ
];
```

#### app.php
```php
return [
    'debug' => true,  // ডিবাগ মোড, ত্রুটিতে স্ট্যাক ট্রেস ইত্যাদি সক্ষম করে। প্রোডাকশনে নিষ্ক্রিয় করা উচিত
    'error_reporting' => E_ALL, // ত্রুটি রিপোর্টিং স্তর
    'default_timezone' => 'Asia/Shanghai', // ডিফল্ট টাইমজোন
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // পাবলিক ডিরেক্টরি পাথ
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // রানটাইম ডিরেক্টরি পাথ
    'controller_suffix' => 'Controller', // কন্ট্রোলার সাফিক্স
    'controller_reuse' => false, // কন্ট্রোলার পুনঃব্যবহার করা হবে কিনা
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // webman প্রসেস কনফিগারেশন
    'webman' => [ 
        'handler' => Http::class, // প্রসেস হ্যান্ডলার ক্লাস
        'listen' => 'http://0.0.0.0:8787', // শোনার ঠিকানা
        'count' => cpu_count() * 4, // প্রসেস সংখ্যা, ডিফল্টে 4x CPU
        'user' => '', // প্রসেস ব্যবহারকারী, নিম্ন-অধিকার ব্যবহারকারী ব্যবহার করা উচিত
        'group' => '', // প্রসেস গ্রুপ, নিম্ন-অধিকার গ্রুপ ব্যবহার করা উচিত
        'reusePort' => false, // reusePort সক্ষম করুন, worker প্রসেসে সংযোগ বিতরণ করে
        'eventLoop' => '', // ইভেন্ট লুপ ক্লাস, খালি থাকলে server.event_loop ব্যবহার করে
        'context' => [], // শোনার প্রসঙ্গ, যেমন SSL
        'constructor' => [ // প্রসেস হ্যান্ডলারের কনস্ট্রাক্টর প্যারামিটার, এখানে Http ক্লাস
            'requestClass' => Request::class, // কাস্টম রিকোয়েস্ট ক্লাস
            'logger' => Log::channel('default'), // লগার ইনস্ট্যান্স
            'appPath' => app_path(), // অ্যাপ ডিরেক্টরি পাথ
            'publicPath' => public_path() // পাবলিক ডিরেক্টরি পাথ
        ]
    ],
    // ফাইল পরিবর্তন অটো রিলোড এবং মেমরি লিক সনাক্তকরণের জন্য মনিটর প্রসেস
    'monitor' => [
        'handler' => app\process\Monitor::class, // হ্যান্ডলার ক্লাস
        'reloadable' => false, // এই প্রসেস রিলোড চালায় না
        'constructor' => [ // প্রসেস হ্যান্ডলারের কনস্ট্রাক্টর প্যারামিটার
            // পর্যবেক্ষণ করতে ডিরেক্টরি, বেশি রাখবেন না কারণ সনাক্তকরণ মন্থর করে
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // পরিবর্তন পর্যবেক্ষণ করতে ফাইল এক্সটেনশন
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // অন্যান্য অপশন
            'options' => [
                // ফাইল মনিটরিং সক্ষম করুন, শুধুমাত্র Linux, ডেমন মোডে ফাইল মনিটরিং ডিফল্টে নিষ্ক্রিয়
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // মেমরি মনিটরিং সক্ষম করুন, শুধুমাত্র Linux
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// PSR-11 ডিপেন্ডেন্সি ইনজেকশন কনটেইনার ইনস্ট্যান্স ফেরত দিন
return new Webman\Container;
```

#### dependence.php
```php
// ডিপেন্ডেন্সি ইনজেকশন কনটেইনারে সেবা এবং নির্ভরতা কনফিগার করুন
return [];
```

#### route.php
```php

use support\Route;
// /test পাথের জন্য রাউট সংজ্ঞায়িত করুন
Route::any('/test', function (Request $request) {
    return response('test');
});
```

#### view.php
```php
use support\view\Raw;
use support\view\Twig;
use support\view\Blade;
use support\view\ThinkPHP;

return [
    'handler' => Raw::class // ডিফল্ট ভিউ হ্যান্ডলার ক্লাস
];
```

### autoload.php
```php
// ফ্রেমওয়ার্ক অটোলোড ফাইল কনফিগার করুন
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php',
        base_path() . '/support/Response.php',
    ]
];
```

#### cache.php
```php
// ক্যাশ কনফিগারেশন
return [
    'default' => 'file', // ডিফল্ট ড্রাইভার
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // ক্যাশ ফাইল স্টোরেজ পাথ
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // Redis সংযোগ নাম, redis.php এ কনফিগারেশন বোঝায়
        ],
        'array' => [
            'driver' => 'array' // মেমরিতে ক্যাশ, রিস্টার্টে মুছে যায়
        ]
    ]
];
```

#### redis.php
```php
return [
    'default' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 0,
    ],
];
```

#### database.php
```php
return [
 // ডিফল্ট ডাটাবেস
 'default' => 'mysql',
 // ডাটাবেস সংযোগ কনফিগারেশন
 'connections' => [

     'mysql' => [
         'driver'      => 'mysql',
         'host'        => '127.0.0.1',
         'port'        => 3306,
         'database'    => 'webman',
         'username'    => 'webman',
         'password'    => '',
         'unix_socket' => '',
         'charset'     => 'utf8',
         'collation'   => 'utf8_unicode_ci',
         'prefix'      => '',
         'strict'      => true,
         'engine'      => null,
     ],

     'sqlite' => [
         'driver'   => 'sqlite',
         'database' => '',
         'prefix'   => '',
     ],

     'pgsql' => [
         'driver'   => 'pgsql',
         'host'     => '127.0.0.1',
         'port'     => 5432,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
         'schema'   => 'public',
         'sslmode'  => 'prefer',
     ],

     'sqlsrv' => [
         'driver'   => 'sqlsrv',
         'host'     => 'localhost',
         'port'     => 1433,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
     ],
 ],
];
```

#### exception.php
```php
return [
    // এক্সেপশন হ্যান্ডলার ক্লাস সেট করুন
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // হ্যান্ডলার
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // লগ ফাইল নাম
                    7, //$maxFiles // 7 দিন লগ রাখুন
                    Monolog\Logger::DEBUG, // লগ স্তর
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // ফরম্যাটার
                    'constructor' => [null, 'Y-m-d H:i:s', true], // ফরম্যাটার প্যারামিটার
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // টাইপ
    'type' => 'file', // অথবা redis অথবা redis_cluster
     // হ্যান্ডলার
    'handler' => FileSessionHandler::class,
     // কনফিগারেশন
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // স্টোরেজ ডিরেক্টরি
        ],
        'redis' => [
            'host' => '127.0.0.1',
            'port' => 6379,
            'auth' => '',
            'timeout' => 2,
            'database' => '',
            'prefix' => 'redis_session_',
        ],
        'redis_cluster' => [
            'host' => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth' => '',
            'prefix' => 'redis_session_',
        ]
    ],
    'session_name' => 'PHPSID', // সেশন নাম
    'auto_update_timestamp' => false, // সেশন মেয়াদ উত্তীর্ণ প্রতিরোধে অটো টাইমস্ট্যাম্প আপডেট
    'lifetime' => 7*24*60*60, // আয়ু
    'cookie_lifetime' => 365*24*60*60, // কুকি আয়ু
    'cookie_path' => '/', // কুকি পাথ
    'domain' => '', // কুকি ডোমেইন
    'http_only' => true, // শুধুমাত্র HTTP
    'secure' => false, // শুধুমাত্র HTTPS
    'same_site' => '', // SameSite অ্যাট্রিবিউট
    'gc_probability' => [1, 1000], // সেশন গারবেজ কালেকশন সম্ভাবনা
];
```

#### middleware.php
```php
// মিডলওয়্যার কনফিগার করুন
return [];
```

#### static.php
```php
return [
    'enable' => true, // webman স্ট্যাটিক ফাইল সার্ভিং সক্ষম করুন
    'middleware' => [ // স্ট্যাটিক ফাইল মিডলওয়্যার, ক্যাশ পলিসি, CORS ইত্যাদির জন্য
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // ডিফল্ট ভাষা
    'locale' => 'zh_CN',
    // ফ্যালব্যাক ভাষা
    'fallback_locale' => ['zh_CN', 'en'],
    // অনুবাদ ফাইল অবস্থান
    'path' => base_path() . '/resource/translations',
];
```
