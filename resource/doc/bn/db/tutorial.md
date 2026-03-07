# ডাটাবেস দ্রুত শুরু (Laravel ডাটাবেস কম্পোনেন্ট ভিত্তিক)

[webman/database](https://github.com/webman-php/database) [illuminate/database](https://github.com/illuminate/database) এর উপর ভিত্তি করে তৈরি এবং কানেকশন পুল ফিচার যুক্ত। এটি কোরুটিন ও নন-কোরুটিন উভয় পরিবেশে কাজ করে। ব্যবহার লারাভেলের মতোই।

আপনি [অন্যান্য ডাটাবেস কম্পোনেন্ট ব্যবহার](others.md) বিভাগ দেখে ThinkPHP বা অন্যান্য ডাটাবেস ব্যবহার করতে পারেন।

## ডাটাবেস ইনস্টলেশন

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

ইনস্টলেশনের পর রিস্টার্ট করতে হবে (রিলোড করলে হবে না)।

> **পরামর্শ**
> webman/database লারাভেলের `illuminate/database` এর উপর নির্ভরশীল, তাই ইনস্টলের সময় `illuminate/database` এর ডিপেন্ডেন্সি স্বয়ংক্রিয়ভাবে ইনস্টল হয়।

> **দ্রষ্টব্য**
> পেজিনেশন, ডাটাবেস ইভেন্ট বা SQL লগিং দরকার না হলে শুধু চালান:
> `composer require -W webman/database`

## ডাটাবেস কনফিগারেশন
`config/database.php`
```php

return [
    // ডিফল্ট ডাটাবেস
    'default' => 'mysql',

    // কানেকশন কনফিগারেশন
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'test',
            'username'    => 'root',
            'password'    => '',
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
            'options' => [
                PDO::ATTR_EMULATE_PREPARES => false, // swoole বা swow ব্যবহার করলে অপরিহার্য
            ],
            'pool' => [ // কানেকশন পুল কনফিগারেশন
                'max_connections' => 5, // সর্বোচ্চ কানেকশন সংখ্যা
                'min_connections' => 1, // সর্বনিম্ন কানেকশন সংখ্যা
                'wait_timeout' => 3,    // পুল থেকে কানেকশন নেওয়ার সর্বোচ্চ অপেক্ষার সময়; অতিক্রম করলে এক্সেপশন। শুধু কোরুটিন পরিবেশে কার্যকর
                'idle_timeout' => 60,   // পুলে কানেকশনের সর্বোচ্চ নিষ্ক্রিয় সময়; অতিক্রমে বন্ধ হয়ে min_connections পর্যন্ত কমে
                'heartbeat_interval' => 50, // পুল হৃদস্পন্দন ব্যবধানের সময় (সেকেন্ড); ৬০ এর কম রাখা ভাল
            ],
        ],
    ],
];
```

`pool` কনফিগারেশন ছাড়া বাকি সব লারাভেলের মতো।

## কানেকশন পুল সম্পর্কে
* প্রতিটি প্রসেসের নিজস্ব পুল আছে; পুল প্রসেসগুলোর মধ্যে ভাগ হয় না।
* কোরুটিন বন্ধ থাকলে রিকোয়েস্টগুলো ক্রমানুসারে চলে, সমসময়িকতা নেই, তাই পুলে সর্বোচ্চ একটি কানেকশন থাকে।
* কোরুটিন চালু থাকলে রিকোয়েস্টগুলো একসাথে চলে; পুল চাহিদা অনুযায়ী কানেকশন সংখ্যা বাড়ায়-কমায়, `max_connections` ছাড়িয়ে যায় না, `min_connections` এর নিচেও নামে না।
* পুলে সর্বোচ্চ `max_connections` হওয়ায়, ডাটাবেস ব্যবহারকারী কোরুটিন এর বেশি হলে কিছু `wait_timeout` সেকেন্ড পর্যন্ত অপেক্ষায় থাকে; অতিক্রম করলে এক্সেপশন হয়।
* অলস অবস্থায় (কোরুটিন সহ বা ছাড়া) কানেকশন `idle_timeout` পরে ফেরত নেওয়া হয়, `min_connections` পর্যন্ত (`min_connections` শূন্য হতে পারে)।

## ডাটাবেস ব্যবহারের উদাহরণ
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        $default_uid = 29;
        $uid = $request->get('uid', $default_uid);
        $name = Db::table('users')->where('uid', $uid)->value('username');
        return response("hello $name");
    }
}
```

ব্যবহার লারাভেলের মতোই; `Db::table()` মেথড দিয়ে ডাটাবেস অপারেশন করা হয়।
