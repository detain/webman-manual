# Redis কিউ

রেডিস ভিত্তিক মেসেজ কিউ, বিলম্বিত মেসেজ প্রক্রিয়াকরণ সমর্থন করে।

## ইন্সটলেশন
`composer require webman/redis-queue`

## কনফিগারেশন ফাইল
রেডিস কনফিগারেশন ফাইল `{মূল-প্রকল্প}/config/plugin/webman/redis-queue/redis.php` এ স্বয়ংক্রিয়ভাবে তৈরি হয়, এর বিষয়বস্তু নিম্নোক্তের মতো:
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // পাসওয়ার্ড, ঐচ্ছিক
            'db' => 0,            // ডাটাবেস
            'max_attempts'  => 5, // ব্যবহার ব্যর্থ হলে পুনরায় চেষ্টার সংখ্যা
            'retry_seconds' => 5, // পুনরায় চেষ্টার ব্যবধান, সেকেন্ডে
        ]
    ],
];
```

### ব্যবহার ব্যর্থ হলে পুনরায় চেষ্টা
যদি ব্যবহার ব্যর্থ হয় (এক্সসেপশন ঘটে), তাহলে মেসেজ বিলম্বিত কিউতে রাখা হবে এবং পরবর্তী পুনরায় চেষ্টার জন্য অপেক্ষা করবে। পুনরায় চেষ্টার সংখ্যা `max_attempts` দ্বারা নিয়ন্ত্রিত, এবং ব্যবধান `retry_seconds` ও `max_attempts` দ্বারা যৌথভাবে নিয়ন্ত্রিত। যেমন, `max_attempts` ৫ এবং `retry_seconds` ১০ হলে, প্রথম পুনরায় চেষ্টার ব্যবধান `1*10` সেকেন্ড, দ্বিতীয়টি `2*10` সেকেন্ড, তৃতীয়টি `3*10` সেকেন্ড, ইত্যাদি ৫টি পুনরায় চেষ্টা পর্যন্ত। যদি `max_attempts` সেট করা পুনরায় চেষ্টার সংখ্যা অতিক্রম করে, তাহলে মেসেজ `{redis-queue}-failed` কী সহ ব্যর্থ কিউতে রাখা হবে।

## মেসেজ প্রেরণ (সিনক্রোনাস)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // কিউর নাম
        $queue = 'send-mail';
        // ডেটা, সরাসরি অ্যারে পাঠানো যাবে, সিরিয়ালাইজ করার দরকার নেই
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // মেসেজ প্রেরণ
        Redis::send($queue, $data);
        // বিলম্বিত মেসেজ প্রেরণ, ৬০ সেকেন্ড পর প্রক্রিয়া হবে
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
প্রেরণ সফল হলে `Redis::send()` true রিটার্ন করে, নাহলে false বা এক্সসেপশন।

> **সুচনা**
> বিলম্বিত কিউ ব্যবহারের সময়ে বিচ্যুতি হতে পারে। যেমন, ব্যবহারের গতি উৎপাদনের গতির চেয়ে ধীর হলে কিউ জমা হয় এবং ব্যবহার বিলম্বিত হয়। প্রতিকার: আরো কনজিউমার প্রসেস চালানো।

## মেসেজ প্রেরণ (অ্যাসিনক্রোনাস)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // কিউর নাম
        $queue = 'send-mail';
        // ডেটা, সরাসরি অ্যারে পাঠানো যাবে, সিরিয়ালাইজ করার দরকার নেই
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // মেসেজ প্রেরণ
        Client::send($queue, $data);
        // বিলম্বিত মেসেজ প্রেরণ, ৬০ সেকেন্ড পর প্রক্রিয়া হবে
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` কোন মান রিটার্ন করে না। এটি অ্যাসিনক্রোনাস পুশ এবং Redis-এ ১০০% মেসেজ ডেলিভারির নিশ্চয়তা দেয় না।

> **সুচনা**
> `Client::send()`-এর নীতি হলো স্থানীয় মেমোরিতে মেমোরি কিউ তৈরি করা এবং অ্যাসিনক্রোনাসে Redis-এ মেসেজ সিঙ্ক করা (সিঙ্ক দ্রুত, প্রায় সেকেন্ডে ১০,০০০ মেসেজ)। যদি প্রসেস রিস্টার্ট হয় এবং স্থানীয় মেমোরি কিউ-এর ডেটা পুরো সিঙ্ক না হয়, তাহলে মেসেজ হারানো হতে পারে। `Client::send()` অ্যাসিনক্রোনাস প্রেরণ অগুরুত্বপূর্ণ মেসেজের জন্য উপযুক্ত।

> **সুচনা**
> `Client::send()` অ্যাসিনক্রোনাস এবং শুধু Workerman রানটাইমে ব্যবহার করা যাবে। কমান্ড লাইনে সিনক্রোনাস ইন্টারফেস `Redis::send()` ব্যবহার করুন।

## অন্যান্য প্রকল্প থেকে মেসেজ প্রেরণ
কখনো কখনো আপনাকে অন্য প্রকল্প থেকে মেসেজ প্রেরণ করতে হবে এবং `webman\redis-queue` ব্যবহার করতে পারবেন না। এমন ক্ষেত্রে নিচের ফাংশন অনুসরণ করে কিউতে মেসেজ প্রেরণ করতে পারেন।

```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting';
    $queue_delay = '{redis-queue}-delayed';
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```

এখানে `$redis` প্যারামিটার Redis ইন্সট্যান্স। যেমন redis এক্সটেনশন ব্যবহার এরকম:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## ব্যবহার
কনজিউমার প্রসেস কনফিগারেশন ফাইল `{মূল-প্রকল্প}/config/plugin/webman/redis-queue/process.php` এ। কনজিউমার ডিরেক্টরি `{মূল-প্রকল্প}/app/queue/redis/` এর অধীনে।

`php webman redis-queue:consumer my-send-mail` কমান্ড চালালে `{মূল-প্রকল্প}/app/queue/redis/MyMailSend.php` ফাইল তৈরি হবে।

> **সুচনা**
> এই কমান্ডের জন্য [কনসোল](../plugin/console.md) প্লাগইন ইন্সটল করতে হবে। ইন্সটল না চাইলে নিচের মতো ম্যানুয়ালি তৈরি করতে পারেন:

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // ব্যবহার করার কিউর নাম
    public $queue = 'send-mail';

    // সংযোগের নাম, plugin/webman/redis-queue/redis.php-এর সংযোগের সাথে মিল
    public $connection = 'default';

    // ব্যবহার
    public function consume($data)
    {
        // ডিসিরিয়ালাইজ করার দরকার নেই
        var_export($data); // আউটপুট ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // ব্যবহার ব্যর্থ কলব্যাক
    /* 
    $package = [
        'id' => 1357277951, // মেসেজ আইডি
        'time' => 1709170510, // মেসেজ সময়
        'delay' => 0, // বিলম্বের সময়
        'attempts' => 2, // ব্যবহারের সংখ্যা
        'queue' => 'send-mail', // কিউর নাম
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // মেসেজ বিষয়বস্তু
        'max_attempts' => 5, // সর্বোচ্চ পুনরায় চেষ্টা
        'error' => 'এরর মেসেজ' // এরর মেসেজ
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // ডিসিরিয়ালাইজ করার দরকার নেই
        var_export($package); 
    }
}
```

> **দ্রষ্টব্য**
> ব্যবহারের সময় কোন এক্সসেপশন বা Error না ছুঁড়ে দিলে ব্যবহার সফল ধরা হয়; নাহলে ব্যর্থ এবং মেসেজ পুনরায় চেষ্টা কিউতে যায়। redis-queue-এর ack মেকানিজম নেই; একে অটো ack হিসেবে দেখতে পারেন (কোন এক্সসেপশন বা Error না হলে)। বর্তমান মেসেজ সফলভাবে ব্যবহার হয়নি চিহ্নিত করতে চাইলে ম্যানুয়ালি এক্সসেপশন ছুঁড়ে দিয়ে পুনরায় চেষ্টা কিউতে পাঠাতে পারেন। বাস্তবে এটা ack মেকানিজমের মতই।

> **সুচনা**
> কনজিউমাররা একাধিক সার্ভার ও প্রসেস সমর্থন করে, এবং একই মেসেজ **দুইবার** ব্যবহার হবে না। ব্যবহৃত মেসেজ কিউ থেকে অটো মুছে যায়; ম্যানুয়াল মুছে ফেলার দরকার নেই।

> **সুচনা**
> কনজিউমার প্রসেস একসাথে একাধিক ভিন্ন কিউ ব্যবহার করতে পারে। নতুন কিউ যোগ করতে `process.php`-এর কনফিগারেশন বদলানোর দরকার নেই। নতুন কিউ কনজিউমার যোগ করতে `app/queue/redis` এর অধীনে সংশ্লিষ্ট `Consumer` ক্লাস যোগ করুন, এবং `$queue` প্রপার্টি দিয়ে ব্যবহার করবে এমন কিউর নাম নির্দিষ্ট করুন।

> **সুচনা**
> উইন্ডোজ ব্যবহারকারীদের webman চালু করতে `php windows.php` চালাতে হবে, নাহলে কনজিউমার প্রসেস স্টার্ট হবে না।

> **সুচনা**
> onConsumeFailure কলব্যাক প্রতিবার ব্যবহার ব্যর্থ হলে ট্রিগার হয়। ব্যর্থতার পর লজিক এখানে handle করতে পারেন। (এই বৈশিষ্ট্যের জন্য `webman/redis-queue>=1.3.2` এবং `workerman/redis-queue>=1.2.1` প্রয়োজন)

## ভিন্ন কিউর জন্য ভিন্ন কনজিউমার প্রসেস সেট করা
ডিফল্টে সব কনজিউমার একই কনজিউমার প্রসেস ভাগ করে। কিন্তু কখনো কিছু কিউর ব্যবহার আলাদা করতে চাই—যেমন ধীর ব্যবহারের ব্যবসা এক দল প্রসেসে, দ্রুত ব্যবহারের ব্যবসা অন্য দলে। এর জন্য কনজিউমার দু’টি ডিরেক্টরিতে ভাগ করতে পারি, যেমন `app_path() . '/queue/redis/fast'` এবং `app_path() . '/queue/redis/slow'` (নোট: কনজিউমার ক্লাসের namespace সংশ্লিষ্টভাবে আপডেট করতে হবে)। কনফিগারেশন নিম্নরূপ:
```php
return [
    ...অন্য কনফিগারেশন বাদ...
    
    'redis_consumer_fast'  => [ // কী কাস্টম, কোন বিন্যাস বাধ্যতা নেই, এখানে redis_consumer_fast নাম দেওয়া হয়েছে
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // কনজিউমার ক্লাস ডিরেক্টরি
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // কী কাস্টম, কোন বিন্যাস বাধ্যতা নেই, এখানে redis_consumer_slow নাম দেওয়া হয়েছে
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // কনজিউমার ক্লাস ডিরেক্টরি
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

এভাবে দ্রুত ব্যবসার কনজিউমার `queue/redis/fast` ডিরেক্টরিতে এবং ধীর ব্যবসার কনজিউমার `queue/redis/slow` তে যায়, কিউর জন্য কনজিউমার প্রসেস নির্ধারণের উদ্দেশ্য পূরণ হয়।

## একাধিক Redis কনফিগারেশন
#### কনফিগারেশন
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // পাসওয়ার্ড, স্ট্রিং টাইপ, ঐচ্ছিক
            'db' => 0,            // ডাটাবেস
            'max_attempts'  => 5, // ব্যবহার ব্যর্থ হলে পুনরায় চেষ্টার সংখ্যা
            'retry_seconds' => 5, // পুনরায় চেষ্টার ব্যবধান, সেকেন্ডে
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // পাসওয়ার্ড, স্ট্রিং টাইপ, ঐচ্ছিক
            'db' => 0,           // ডাটাবেস
            'max_attempts'  => 5, // ব্যবহার ব্যর্থ হলে পুনরায় চেষ্টার সংখ্যা
            'retry_seconds' => 5, // পুনরায় চেষ্টার ব্যবধান, সেকেন্ডে
        ]
    ],
];
```

নোট: কনফিগারেশনে `other` কী সহ একটি অতিরিক্ত Redis কনফিগারেশন যোগ হয়েছে।

#### একাধিক Redis-এ মেসেজ প্রেরণ

```php
// `default` কী সহ কিউতে মেসেজ প্রেরণ
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// একই
Client::send($queue, $data);
Redis::send($queue, $data);

// `other` কী সহ কিউতে মেসেজ প্রেরণ
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### একাধিক Redis থেকে ব্যবহার
কনফিগারেশনে `other` কী সহ কিউ থেকে মেসেজ ব্যবহার:
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // ব্যবহার করার কিউর নাম
    public $queue = 'send-mail';

    // === কনফিগে other কী সহ কিউ থেকে ব্যবহারের জন্য এখানে 'other' সেট করুন ===
    public $connection = 'other';

    // ব্যবহার
    public function consume($data)
    {
        // ডিসিরিয়ালাইজ করার দরকার নেই
        var_export($data);
    }
}
```

## সাধারণ প্রশ্ন

**কেন `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)` এরর হয়?**

এই এরর শুধু অ্যাসিনক্রোনাস প্রেরণ ইন্টারফেস `Client::send()`-এ হয়। অ্যাসিনক্রোনাস প্রেরণ প্রথমে মেসেজ স্থানীয় মেমোরিতে সেভ করে, তারপর প্রসেস idle হলে Redis-এ পাঠায়। যদি Redis মেসেজ উৎপাদনের গতির চেয়ে ধীরে গ্রহণ করে, অথবা প্রসেস অন্য কাজে ব্যস্ত থাকে এবং মেমোরি থেকে Redis-এ মেসেজ সিঙ্ক করার পর্যাপ্ত সময় না পায়, তাহলে মেসেজ জমা হতে পারে। ৬০০ সেকেন্ডের বেশি মেসেজ জমা থাকলে এই এরর ট্রিগার হয়।

সমাধান: মেসেজ প্রেরণের জন্য সিনক্রোনাস প্রেরণ ইন্টারফেস `Redis::send()` ব্যবহার করুন।
