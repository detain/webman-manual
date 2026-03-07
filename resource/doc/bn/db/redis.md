# রেডিস

[webman/redis](https://github.com/webman-php/redis) [illuminate/redis](https://github.com/illuminate/redis) এর উপর সংযোগ পুল যোগ করে প্রসারিত করে এবং করউটিন ও অকোরউটিন উভয় পরিবেশে কাজ করে। ব্যবহার লারাভেলের মতোই।

`illuminate/redis` ব্যবহারের আগে `php-cli` এর জন্য রেডিস এক্সটেনশন ইনস্টল করা প্রয়োজন।

## ইনস্টলেশন

```php
composer require -W webman/redis illuminate/events
```

ইনস্টলেশনের পরে রিস্টার্ট প্রয়োজন (রিলোড কাজ করে না)।

## কনফিগারেশন

রেডিস কনফিগারেশন ফাইল `config/redis.php` এ আছে:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // সংযোগ পুল সেটিংস
            'max_connections' => 10,     // পুলে সর্বোচ্চ সংযোগ সংখ্যা
            'min_connections' => 1,      // পুলে সর্বনিম্ন সংযোগ সংখ্যা
            'wait_timeout' => 3,         // সংযোগ নিতে সর্বোচ্চ অপেক্ষার সময় (সেকেন্ড)
            'idle_timeout' => 50,        // এর পরে সংযোগ min_connections পর্যন্ত মুক্তি পায়
            'heartbeat_interval' => 50,  // হার্টবিট ব্যবধান (৬০ সেকেন্ডের বেশি নয়)
        ],
    ]
];
```

## সংযোগ পুল সম্পর্কে

* প্রতিটি প্রসেসের নিজস্ব পুল থাকে; পুল প্রসেসের মধ্যে ভাগাভাগি হয় না।
* করউটিন ছাড়া নির্বাহ অনুক্রমিক হয়, তাই সর্বোচ্চ একটি সংযোগ ব্যবহৃত হয়।
* করউটিন সহ নির্বাহ সমবর্তী হয় এবং পুল `min_connections` থেকে `max_connections` পর্যন্ত গতিশীলভাবে সমন্বয় হয়।
* যখন রেডিস ব্যবহারকারী করউটিন সংখ্যা `max_connections` ছাড়িয়ে যায়, তারা সর্বোচ্চ `wait_timeout` সেকেন্ড অপেক্ষা করে; তারপর এক্সেপশন নিক্ষিপ্ত হয়।
* নিষ্ক্রিয় অবস্থায় (করউটিন থাক বা না থাক) সংযোগ `idle_timeout` পর `min_connections` না হওয়া পর্যন্ত মুক্তি পায় (০ বৈধ)।

## উদাহরণ

```php
<?php
namespace app\controller;

use support\Request;
use support\Redis;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Redis::set($key, rand());
        return response(Redis::get($key));
    }
}
```

## রেডিস ইন্টারফেস

```php
Redis::append($key, $value)
Redis::bitCount($key)
Redis::decr($key, $value)
Redis::decrBy($key, $value)
Redis::get($key)
Redis::getBit($key, $offset)
Redis::getRange($key, $start, $end)
Redis::getSet($key, $value)
Redis::incr($key, $value)
Redis::incrBy($key, $value)
Redis::incrByFloat($key, $value)
Redis::mGet(array $keys)
Redis::getMultiple(array $keys)
Redis::mSet($pairs)
Redis::mSetNx($pairs)
Redis::set($key, $value, $expireResolution = null, $expireTTL = null, $flag = null)
Redis::setBit($key, $offset, $value)
Redis::setEx($key, $ttl, $value)
Redis::pSetEx($key, $ttl, $value)
Redis::setNx($key, $value)
Redis::setRange($key, $offset, $value)
Redis::strLen($key)
Redis::del(...$keys)
Redis::exists(...$keys)
Redis::expire($key, $ttl)
Redis::expireAt($key, $timestamp)
Redis::select($dbIndex)
```

এর সমতুল্য:

```php
$redis = Redis::connection('default');
$redis->append($key, $value)
$redis->bitCount($key)
$redis->decr($key, $value)
$redis->decrBy($key, $value)
$redis->get($key)
$redis->getBit($key, $offset)
...
```

> **নোট**
> `Redis::select($db)` ইন্টারফেস সাবধানে ব্যবহার করুন। ওয়েবম্যান মেমোরিতে স্থায়ী ফ্রেমওয়ার্ক হওয়ায় একটি রিকোয়েস্টে ডাটাবেস পরিবর্তন পরবর্তী রিকোয়েস্টকে প্রভাবিত করে। একাধিক ডাটাবেসের জন্য প্রতিটি `$db` এর জন্য আলাদা রেডিস সংযোগ কনফিগার করার পরামর্শ দেওয়া হয়।

## একাধিক রেডিস সংযোগ ব্যবহার

`config/redis.php` কনফিগারেশন ফাইলে উদাহরণ:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
    ],

    'cache' => [
        'host'     => '127.0.0.1',
        'password' => null,
        'port'     => 6379,
        'database' => 1,
    ],

]
```

ডিফল্টভাবে `default` এর অধীনে কনফিগার করা সংযোগ ব্যবহৃত হয়। কোন রেডিস সংযোগ ব্যবহার করতে হবে তা বাছাই করতে `Redis::connection()` মেথড ব্যবহার করুন:

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## ক্লাস্টার কনফিগারেশন

আপনার অ্যাপ Redis সার্ভার ক্লাস্টার ব্যবহার করলে কনফিগারেশনে `clusters` কী এর অধীনে সংজ্ঞায়িত করুন:

```php
return [
    'clusters' => [
        'default' => [
            [
                'host'     => 'localhost',
                'username' => null,
                'password' => null,
                'port'     => 6379,
                'database' => 0,
            ],
        ],
    ],

];
```

ডিফল্টভাবে ক্লাস্টার নোডে ক্লায়েন্ট-সাইড শার্ডিং করতে পারে, যাতে নোড পুল এবং প্রচুর মেমোরি সম্ভব হয়। খেয়াল রাখুন ক্লায়েন্ট শার্ডিং ব্যর্থতা সামলায় না; তাই এটি মূলত অন্য প্রধান ডাটাবেস থেকে ক্যাশ ডেটার জন্য উপযুক্ত। Redis নেটিভ ক্লাস্টারের জন্য কনফিগারেশনে `options` এর অধীনে নির্দিষ্ট করুন:

```php
return[
    'options' => [
        'cluster' => 'redis',
    ],

    'clusters' => [
        // ...
    ],
];
```

## পাইপলাইন কমান্ড

যখন একটি অপারেশনে সার্ভারে অনেক কমান্ড পাঠাতে হবে তখন পাইপলাইন ব্যবহার করুন। `pipeline` মেথড একটি ক্লোজার গ্রহণ করে; সমস্ত কমান্ড একটি অপারেশনে নির্বাহিত হয়:

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
