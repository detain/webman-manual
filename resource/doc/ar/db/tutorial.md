# البدء السريع بقاعدة البيانات (بناءً على مكون Laravel)

[webman/database](https://github.com/webman-php/database) مبني على [illuminate/database](https://github.com/illuminate/database) مع إضافة تجميع الاتصالات، ويدعم بيئة الكوروتين وغير الكوروتين. الاستخدام مطابق لـ Laravel.

يمكنك أيضاً الرجوع إلى [استخدام مكونات قاعدة بيانات أخرى](others.md) لاستخدام ThinkPHP أو قواعد بيانات أخرى.

## تثبيت قاعدة البيانات

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

يجب إعادة تشغيل التطبيق بعد التثبيت (reload لن ينجح).

> **تنبيه**
> webman/database يعتمد على `illuminate/database` في Laravel، لذلك سيتم تثبيت حزم الاعتماديات تلقائياً.

> **ملاحظة**
> إذا لم تكن بحاجة إلى التصفح، أحداث قاعدة البيانات، أو تسجيل SQL، يكفي تنفيذ:
> `composer require -W webman/database`

## إعداد قاعدة البيانات
`config/database.php`
```php

return [
    // قاعدة البيانات الافتراضية
    'default' => 'mysql',

    // إعدادات الاتصال بقاعدة البيانات
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
                PDO::ATTR_EMULATE_PREPARES => false, // مطلوب عند استخدام swoole أو swow كبيئة تشغيل
            ],
            'pool' => [ // إعداد تجميع الاتصالات
                'max_connections' => 5, // الحد الأقصى لعدد الاتصالات
                'min_connections' => 1, // الحد الأدنى لعدد الاتصالات
                'wait_timeout' => 3,    // أقصى وقت انتظار للحصول على اتصال من المجمع؛ يتسبب في استثناء عند الانتهاء. فعال فقط في بيئة الكوروتين
                'idle_timeout' => 60,   // أقصى وقت خمول للاتصالات؛ تُغلق وتُسترد بعد انتهائه حتى تصل إلى min_connections
                'heartbeat_interval' => 50, // فاصل نبض المجمع بالثواني، يُنصح بأن يكون أقل من 60
            ],
        ],
    ],
];
```

ما عدا إعدادات `pool`، الباقي مطابق لـ Laravel.

## حول تجميع الاتصالات
* لكل عملية مجمع اتصالات خاص؛ المجمعات لا تُشارك بين العمليات.
* عند عدم استخدام الكوروتين، الطلبات تُنفّذ بالتسلسل داخل العملية، فلا يوجد تنفيذ متزامن، وبالتالي لا يتجاوز المجمع اتصالاً واحداً.
* مع الكوروتين، الطلبات تُنفّذ بالتزامن؛ المجمع يضبط عدد الاتصالات ديناميكياً بحيث لا يتجاوز `max_connections` ولا يقل عن `min_connections`.
* نظراً لأن حد المجمع هو `max_connections`، عند تجاوز عدد الكوروتينات التي تستخدم قاعدة البيانات هذا الحد، ستنتظر بعضها في الطابور حتى `wait_timeout` ثانية؛ التجاوز يُطلق استثناءً.
* في حالة الخمول (مع الكوروتين وبدونه)، تُسترد الاتصالات بعد `idle_timeout` حتى يصل العدد إلى `min_connections` (يمكن أن يكون `min_connections` صفراً).


## مثال على استخدام قاعدة البيانات
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

الاستخدام مطابق لـ Laravel: استخدم الطريقة `Db::table()` للتعامل مع قاعدة البيانات.
