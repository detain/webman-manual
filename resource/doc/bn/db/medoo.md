# Medoo ডাটাবেস

[webman/medoo](https://github.com/webman-php/medoo) [Medoo](https://medoo.in/) কে কানেকশন পুল সহায়তার সাথে প্রসারিত করে এবং করাউটিন ও অ-করাউটিন উভয় পরিবেশে কাজ করে। ব্যবহার Medoo-এর মতোই।

## ইনস্টলেশন
`composer require webman/medoo`

## Medoo ডাটাবেস কনফিগারেশন
কনফিগারেশন ফাইলের অবস্থান: `config/plugin/webman/medoo/database.php`

## Medoo ডাটাবেস ব্যবহার
```php
<?php
namespace app\controller;

use support\Request;
use support\Medoo;

class Index
{
    public function index(Request $request)
    {
        $user = Medoo::get('user', '*', ['uid' => 1]);
        return json($user);
    }
}
```

> **পরামর্শ**
> `Medoo::get('user', '*', ['uid' => 1]);`
> এর সমতুল্য
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## Medoo একাধিক ডাটাবেস কনফিগারেশন

**কনফিগারেশন**
`config/plugin/webman/medoo/database.php` এ নতুন কনফিগারেশন যোগ করুন; কী যেকোনো হতে পারে, এখানে `other` ব্যবহার করা হয়েছে।

```php
<?php
return [
    'default' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [ // কানেকশন পুল কনফিগারেশন
            'max_connections' => 5, // সর্বোচ্চ কানেকশন সংখ্যা
            'min_connections' => 1, // সর্বনিম্ন কানেকশন সংখ্যা
            'wait_timeout' => 60,   // পুল থেকে কানেকশন নেওয়ার সর্বোচ্চ অপেক্ষার সময়; সময়সীমা অতিক্রম করলে ব্যতিক্রম
            'idle_timeout' => 3,    // পুলে কানেকশনের সর্বোচ্চ নিষ্ক্রিয় সময়; অতিক্রম করলে বন্ধ হয়ে min_connections পর্যন্ত কমে যায়
            'heartbeat_interval' => 50, // পুল হার্টবিট ব্যবধান (সেকেন্ডে); ৬০ সেকেন্ডের কম সুপারিশ করা হয়
        ]
    ],
    // এখানে 'other' কনফিগারেশন যোগ করুন
    'other' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [
            'max_connections' => 5,
            'min_connections' => 1,
            'wait_timeout' => 60,
            'idle_timeout' => 3,
            'heartbeat_interval' => 50,
        ],
    ],
];
```

## Medoo ডাটাবেস ব্যবহার
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

দেখুন [Medoo অফিশিয়াল ডকুমেন্টেশন](https://medoo.in/api/select)
