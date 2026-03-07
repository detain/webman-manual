# Medoo डेटाबेस

[webman/medoo](https://github.com/webman-php/medoo) [Medoo](https://medoo.in/) को कनेक्शन पूल समर्थन के साथ विस्तारित करता है और कोरूटीन तथा गैर-कोरूटीन दोनों वातावरणों में काम करता है। उपयोग Medoo के समान है।

## स्थापना
`composer require webman/medoo`

## Medoo डेटाबेस विन्यास
विन्यास फ़ाइल स्थान: `config/plugin/webman/medoo/database.php`

## Medoo डेटाबेस उपयोग
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

> **संकेत**
> `Medoo::get('user', '*', ['uid' => 1]);`
> के बराबर है
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## Medoo कई डेटाबेस विन्यास

**विन्यास**
`config/plugin/webman/medoo/database.php` में नया विन्यास जोड़ें; कुंजी कोई भी हो सकती है, यहाँ `other` का उपयोग किया गया है।

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
        'pool' => [ // कनेक्शन पूल विन्यास
            'max_connections' => 5, // अधिकतम कनेक्शन संख्या
            'min_connections' => 1, // न्यूनतम कनेक्शन संख्या
            'wait_timeout' => 60,   // पूल से कनेक्शन प्राप्त करने की अधिकतम प्रतीक्षा समय; समय सीमा पार होने पर अपवाद
            'idle_timeout' => 3,    // पूल में कनेक्शन की अधिकतम निष्क्रिय समय; पार होने पर बंद होकर min_connections तक कम
            'heartbeat_interval' => 50, // पूल हार्टबीट अंतराल (सेकंड में); 60 सेकंड से कम की सिफारिश
        ]
    ],
    // यहाँ 'other' विन्यास जोड़ें
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

## Medoo डेटाबेस उपयोग
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

देखें [Medoo आधिकारिक दस्तावेज़](https://medoo.in/api/select)
