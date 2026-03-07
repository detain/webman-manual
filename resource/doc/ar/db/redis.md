# Redis

[webman/redis](https://github.com/webman-php/redis) يمتد فوق [illuminate/redis](https://github.com/illuminate/redis) بإضافة مجموعة اتصالات، ويعمل في البيئات ذات الـcoroutine وبدونها. الاستخدام مطابق لـ Laravel.

يجب تثبيت امتداد redis لـ `php-cli` قبل استخدام `illuminate/redis`.

## التثبيت

```php
composer require -W webman/redis illuminate/events
```

بعد التثبيت يجب إعادة التشغيل (reload غير فعال).

## الإعداد

ملف إعداد Redis موجود في `config/redis.php`:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // إعدادات مجموعة الاتصالات
            'max_connections' => 10,     // الحد الأقصى للاتصالات في المجموعة
            'min_connections' => 1,      // الحد الأدنى للاتصالات في المجموعة
            'wait_timeout' => 3,         // أقصى مدة انتظار للحصول على اتصال (ثوانٍ)
            'idle_timeout' => 50,        // مدة الخمول؛ تُحرر الاتصالات بعده حتى تصل إلى min_connections
            'heartbeat_interval' => 50,  // فترة نبضات القلب (لا تتجاوز 60 ثانية)
        ],
    ]
];
```

## مجموعة الاتصالات

* لكل عملية مجموعتها الخاصة؛ المجموعات غير مشتركة بين العمليات.
* بدون coroutine يكون التنفيذ تسلسليًا، فتستخدم اتصالاً واحدًا كحد أقصى.
* مع coroutine يكون التنفيذ متوازيًا وتتدرج المجموعة بين `min_connections` و`max_connections`.
* إذا فاق عدد الـcoroutines التي تستخدم Redis قيمة `max_connections`، تنتظر حتى `wait_timeout` ثانية؛ بعدها يُرمى استثناء.
* عند الخمول (مع أو بدون coroutine)، تُحرر الاتصالات بعد `idle_timeout` حتى تصل إلى `min_connections` (صفر مسموح).

## مثال

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

## واجهة Redis

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

مكافئ لـ:

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

> **ملاحظة**
> استخدم `Redis::select($db)` بحذر. لأن webman إطار يعمل في الذاكرة باستمرار، تغيير قاعدة البيانات في طلب واحد يؤثر على الطلبات التالية. لقواعد بيانات متعددة يُفضّل إعداد اتصال Redis منفصل لكل `$db`.

## استخدام عدة اتصالات Redis

مثال في ملف `config/redis.php`:

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

افتراضيًا تُستخدم الاتصال المحدد في `default`. استخدم `Redis::connection()` لاختيار أي اتصال Redis يُستخدم:

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## إعداد المجموعة

إذا كانت التطبيق يستخدم مجموعة خوادم Redis، عرّفها في الإعدادات تحت المفتاح `clusters`:

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

افتراضيًا تُجزّأ المجموعة على مستوى العميل في العقد، مما يسمح بإنشاء مجمعات وذاكرة كبيرة. تجزئة العميل لا تتعامل مع الأعطال؛ لذا تُستخدم أساسًا لبيانات الكاش من قاعدة رئيسية أخرى. لمجموعة Redis الأصلية حدّد ذلك في `options` في الإعدادات:

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

## أوامر Pipeline

عند إرسال الكثير من الأوامر في عملية واحدة، استخدم pipeline. الدالة `pipeline` تقبل closure؛ تُنفَّذ جميع الأوامر دفعة واحدة:

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
