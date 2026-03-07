# ENV কম্পোনেন্ট vlucas/phpdotenv

## বর্ণনা
`vlucas/phpdotenv` একটি পরিবেশ ভেরিয়েবল লোডিং কম্পোনেন্ট, যা বিভিন্ন পরিবেশের (যেমন বিকাশ, পরীক্ষা ইত্যাদি) কনফিগারেশন আলাদা করার জন্য ব্যবহৃত হয়।

## প্রজেক্ট লিঙ্ক

https://github.com/vlucas/phpdotenv
  
## ইনস্টলেশন
 
```php
composer require vlucas/phpdotenv
 ```
  
## ব্যবহার

### প্রজেক্টের রুট ডিরেক্টরিতে নতুন `.env` ফাইল তৈরি করুন
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### কনফিগারেশন ফাইল পরিবর্তন করুন
**config/database.php**
```php
return [
    // ডিফল্ট ডাটাবেস
    'default' => 'mysql',

    // বিভিন্ন ডাটাবেস কনফিগারেশন
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **পরামর্শ**
> `.env` ফাইলটি `.gitignore` তালিকায় যোগ করার পরামর্শ দেওয়া হয়, যাতে কোড রিপোজিটরিতে কমিট না হয়। রিপোজিটরিতে `.env.example` কনফিগারেশন নমুনা ফাইল যোগ করুন। প্রজেক্ট ডিপ্লয় করার সময় `.env.example` কে `.env` হিসেবে কপি করুন এবং বর্তমান পরিবেশ অনুযায়ী `.env` এর কনফিগারেশন পরিবর্তন করুন। এভাবে প্রজেক্ট বিভিন্ন পরিবেশে ভিন্ন কনফিগারেশন লোড করতে পারবে।

> **নোট**
> `vlucas/phpdotenv` PHP TS সংস্করণে (Thread Safe) বাগ থাকতে পারে। অনুগ্রহ করে NTS সংস্করণ (Non-Thread-Safe) ব্যবহার করুন। বর্তমান PHP সংস্করণ `php -v` চালিয়ে পরীক্ষা করা যায়।

## আরও তথ্য

https://github.com/vlucas/phpdotenv এ যান।
  
