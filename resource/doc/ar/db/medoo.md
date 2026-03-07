# قاعدة بيانات Medoo

[webman/medoo](https://github.com/webman-php/medoo) يمتد [Medoo](https://medoo.in/) بدعم مجمع الاتصالات ويعمل في بيئات الروتينات التآزرية وغير التآزرية. الاستخدام مطابق لـ Medoo.

## التثبيت
`composer require webman/medoo`

## تكوين قاعدة بيانات Medoo
موقع ملف التكوين: `config/plugin/webman/medoo/database.php`

## استخدام قاعدة بيانات Medoo
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

> **تلميح**
> `Medoo::get('user', '*', ['uid' => 1]);`
> يعادل
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## تكوين قواعد بيانات Medoo المتعددة

**التكوين**
أضف تكوينًا جديدًا في `config/plugin/webman/medoo/database.php` بأي مفتاح؛ هنا نستخدم `other`.

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
        'pool' => [ // تكوين مجمع الاتصالات
            'max_connections' => 5, // الحد الأقصى لعدد الاتصالات
            'min_connections' => 1, // الحد الأدنى لعدد الاتصالات
            'wait_timeout' => 60,   // أقصى وقت انتظار للحصول على اتصال من المجمع؛ استثناء عند التجاوز
            'idle_timeout' => 3,    // أقصى وقت خمول للاتصالات في المجمع؛ تتوقف عند التجاوز حتى min_connections
            'heartbeat_interval' => 50, // فاصل نبضات المجمع بالثواني؛ يُوصى بأقل من 60 ثانية
        ]
    ],
    // إضافة تكوين 'other' هنا
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

## استخدام قاعدة بيانات Medoo
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

راجع [التوثيق الرسمي لـ Medoo](https://medoo.in/api/select)
