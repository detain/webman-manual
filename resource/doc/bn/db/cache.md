# ক্যাশ

[webman/cache](https://github.com/webman-php/cache) হলো [symfony/cache](https://github.com/symfony/cache) ভিত্তিক একটি ক্যাশ কম্পোনেন্ট, করউটিন ও নন-করউটিন উভয় পরিবেশের সাথে সামঞ্জস্যপূর্ণ এবং কানেকশন পুল সাপোর্ট করে।

## ইন্সটলেশন

```php
composer require -W webman/cache
```

## উদাহরণ
```php
<?php
namespace app\controller;

use support\Request;
use support\Cache;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Cache::set($key, rand());
        return response(Cache::get($key));
    }
}
```

## কনফিগারেশন ফাইল অবস্থান
কনফিগারেশন ফাইলটি `config/cache.php` এ রয়েছে। নেইলে ম্যানুয়ালি তৈরি করুন।

## কনফিগারেশন ফাইল বিষয়বস্তু
```php
<?php
return [
    'default' => 'file',
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default'
        ],
        'array' => [
            'driver' => 'array'
        ],
        'apcu' => [
            'driver' => 'apcu'
        ]
    ]
];
```
`stores.driver` চারটি ড্রাইভার সাপোর্ট করে: **file**, **redis**, **array** এবং **apcu**।

#### file ড্রাইভার
ডিফল্ট ড্রাইভার। বাহ্যিক নির্ভরতা নেই। প্রসেসের মধ্যে ক্যাশ শেয়ারিং সাপোর্ট করে। একাধিক সার্ভারের মধ্যে শেয়ারিং সাপোর্ট করে না।

#### array ড্রাইভার
মেমোরি স্টোরেজ, সেরা পারফরম্যান্স কিন্তু মেমোরি খরচ করে। প্রসেস বা সার্ভারের মধ্যে শেয়ারিং সাপোর্ট করে না। প্রসেস রিস্টার্টে ডেটা হারিয়ে যায়। ছোট ক্যাশ ভলিউমের প্রজেক্টের জন্য উপযুক্ত।

#### apcu ড্রাইভার
মেমোরি স্টোরেজ। পারফরম্যান্স array এর পর দ্বিতীয়। প্রসেসের মধ্যে ক্যাশ শেয়ারিং সাপোর্ট করে। একাধিক সার্ভারের মধ্যে শেয়ারিং সাপোর্ট করে না। প্রসেস রিস্টার্টে ডেটা হারিয়ে যায়। ছোট ক্যাশ ভলিউমের প্রজেক্টের জন্য উপযুক্ত।

> [APCu এক্সটেনশন](https://pecl.php.net/package/APCu) ইন্সটল এবং সক্রিয় থাকা প্রয়োজন; ঘন ঘন ক্যাশ লেখা/মোছার সিনারিওর জন্য উপযুক্ত নয়, পারফরম্যান্স স্পষ্টভাবে কমে যেতে পারে।

#### redis ড্রাইভার
[webman/redis](./redis.md) কম্পোনেন্টের উপর নির্ভরশীল। প্রসেস ও সার্ভারের মধ্যে ক্যাশ শেয়ারিং সাপোর্ট করে।

**stores.redis.connection**

`stores.redis.connection` হলো `config/redis.php` এ ডিফাইন্ড key এর সাথে মিলিত। Redis ব্যবহার করলে `webman/redis` এর কনফিগারেশন পুনরায় ব্যবহৃত হয়, কানেকশন পুল সেটিংস সহ।

**`config/redis.php` এ ক্যাশের জন্য আলাদা Redis কনফিগারেশন যোগ করার পরামর্শ দেওয়া হয়, উদাহরণ:**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== নতুন যোগ
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

তারপর `stores.redis.connection` কে `cache` সেট করুন। চূড়ান্ত `config/cache.php` এমন হবে:

```php
<?php
return [
    'default' => 'redis', // <====
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache' // <====
        ],
        'array' => [
            'driver' => 'array'
        ]
    ]
];
```

## স্টোর স্যুইচ করা
ভিন্ন ড্রাইভার ব্যবহার করতে ম্যানুয়ালি স্টোর স্যুইচ করা যায়, যেমন:

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **টিপ**
> ক্যাশ key নাম [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) দ্বারা সীমাবদ্ধ এবং `{}()/\@:` এর কোনো অক্ষর থাকতে পারবে না। `symfony/cache` 7.2.4 থেকে PHP ini অপশন `zend.assertions=-1` দিয়ে এই চেক সাময়িকভাবে এড়ানো যায়।

## অন্যান্য ক্যাশ কম্পোনেন্ট ব্যবহার

[ThinkCache](https://github.com/webman-php/think-cache) কম্পোনেন্টের জন্য [অন্যান্য ডাটাবেস](others.md#ThinkCache) দেখুন।
