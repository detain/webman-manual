# webman সেশন ব্যবস্থাপনা

## উদাহরণ
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

`$request->session();` দিয়ে `Workerman\Protocols\Http\Session` ইনস্ট্যান্স নিন এবং এর মেথড দিয়ে সেশন ডেটা যোগ, পরিবর্তন বা মুছুন।

> **নোট**
> সেশন অবজেক্ট ধ্বংস হলে সেশন ডেটা স্বয়ংক্রিয়ভাবে সংরক্ষিত হয়।
> সেশন অবজেক্ট গ্লোবাল ভেরিয়েবলে রাখলে তা ধ্বংস হয় না এবং স্বয়ংক্রিয়ভাবে সংরক্ষিত হয় না। এমন ক্ষেত্রে ডেটা সংরক্ষণ করতে `$session->save()` ম্যানুয়ালি কল করুন।

## সব সেশন ডেটা আনুন
```php
$session = $request->session();
$all = $session->all();
```
এটি একটি অ্যারে ফেরত দেয়। কোনো সেশন ডেটা না থাকলে খালি অ্যারে ফেরত দেয়।


## সেশন থেকে একটি মান আনুন
```php
$session = $request->session();
$name = $session->get('name');
```
ডেটা না থাকলে null ফেরত দেয়।

get মেথডের দ্বিতীয় আর্গুমেন্ট হিসেবে ডিফল্ট মান পাঠাতে পারেন। সেশন অ্যারেতে সংগত মান না পেলে ডিফল্ট মান ফেরত দেয়। উদাহরণ:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## সেশন সংরক্ষণ করুন
একটি ডেটা সংরক্ষণ করতে set মেথড ব্যবহার করুন।
```php
$session = $request->session();
$session->set('name', 'tom');
```
set কোনো মান ফেরত দেয় না। সেশন অবজেক্ট ধ্বংস হলে সেশন স্বয়ংক্রিয়ভাবে সংরক্ষিত হয়।

কয়েকটি মান সংরক্ষণ করতে put মেথড ব্যবহার করুন।
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
একইভাবে putও কোনো মান ফেরত দেয় না।

## সেশন ডেটা মুছুন
এক বা একাধিক সেশন ডেটা মুছতে `forget` মেথড ব্যবহার করুন।
```php
$session = $request->session();
// একটি আইটেম মুছুন
$session->forget('name');
// একাধিক আইটেম মুছুন
$session->forget(['name', 'age']);
```

সিস্টেম delete মেথডও দেয়। forget থেকে আলাদা, delete শুধুমাত্র একটি আইটেম মুছতে পারে।
```php
$session = $request->session();
// $session->forget('name'); এর সমতুল্য
$session->delete('name');
```


## সেশন থেকে মান আনুন এবং মুছুন
```php
$session = $request->session();
$name = $session->pull('name');
```
নিচের কোডের সমতুল্য:
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
সংগত সেশন না থাকলে null ফেরত দেয়।


## সব সেশন ডেটা মুছুন
```php
$request->session()->flush();
```
কোনো মান ফেরত দেয় না। সেশন অবজেক্ট ধ্বংস হলে সেশন স্বয়ংক্রিয়ভাবে স্টোরেজ থেকে মুছে যায়।


## সেশন মানের অস্তিত্ব পরীক্ষা করুন
```php
$session = $request->session();
$has = $session->has('name');
```
সেশন মান না থাকলে বা null হলে false ফেরত দেয়; অন্যথায় true ফেরত দেয়।

```php
$session = $request->session();
$has = $session->exists('name');
```
উপরের কোডও সেশন মানের অস্তিত্ব পরীক্ষা করে। পার্থক্য: মান null হলেও `exists` true ফেরত দেয়।

## হেল্পার ফাংশন session()

webman একই কাজের জন্য হেল্পার ফাংশন `session()` সরবরাহ করে।
```php
// সেশন ইনস্ট্যান্স আনুন
$session = session();
// সমতুল্য
$session = $request->session();

// মান আনুন
$value = session('key', 'default');
// সমতুল্য
$value = session()->get('key', 'default');
// সমতুল্য
$value = $request->session()->get('key', 'default');

// সেশনে মান নির্ধারণ করুন
session(['key1'=>'value1', 'key2' => 'value2']);
// সমতুল্য
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// সমতুল্য
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## কনফিগ ফাইল
সেশন কনফিগ ফাইল `config/session.php` এ আছে। বিষয়বস্তু এরকম:
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class বা RedisSessionHandler::class বা RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // handler FileSessionHandler::class হলে মান 'file',
    // handler RedisSessionHandler::class হলে মান 'redis'
    // handler RedisClusterSessionHandler::class হলে মান 'redis_cluster' (Redis ক্লাস্টার)
    'type'    => 'file',

    // বিভিন্ন হ্যান্ডলার বিভিন্ন কনফিগ ব্যবহার করে
    'config' => [
        // type 'file' এর কনফিগ
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // type 'redis' এর কনফিগ
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // session_id সংরক্ষণকারী কুকির নাম
    'auto_update_timestamp' => false,  // সেশন স্বয়ংক্রিয় রিফ্রেশ কিনা, ডিফল্ট: বন্ধ
    'lifetime' => 7*24*60*60,          // সেশন মেয়াদ
    'cookie_lifetime' => 365*24*60*60, // session_id কুকির মেয়াদ
    'cookie_path' => '/',              // session_id কুকির পথ
    'domain' => '',                    // session_id কুকির ডোমেইন
    'http_only' => true,               // httpOnly সক্ষম কিনা, ডিফল্ট: সক্ষম
    'secure' => false,                 // শুধু HTTPS এ সেশন সক্ষম, ডিফল্ট: বন্ধ
    'same_site' => '',                 // CSRF আক্রমণ ও ব্যবহারকারী ট্র্যাকিং রোধে, মান: strict/lax/none
    'gc_probability' => [1, 1000],     // সেশন গারবেজ কালেকশন সম্ভাবনা
];
```

## নিরাপত্তা
সেশনে ক্লাস ইনস্ট্যান্স সরাসরি সংরক্ষণ করতে সুপারিশ করা হয় না, বিশেষত অবিশ্বস্ত উৎস থেকে। ডিসিরিয়ালাইজেশন নিরাপত্তা ঝুঁকি তৈরি করতে পারে।

