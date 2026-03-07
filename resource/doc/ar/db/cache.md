# التخزين المؤقت (Cache)

[webman/cache](https://github.com/webman-php/cache) هو مكون تخزين مؤقت مبني على [symfony/cache](https://github.com/symfony/cache)، متوافق مع البيئات ذات التزامن والغير متزامنة، ويدعم تجمع الاتصالات.

## التثبيت

```php
composer require -W webman/cache
```

## مثال
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

## موقع ملف التكوين
يوجد ملف التكوين في `config/cache.php`. أنشئه يدوياً إن لم يكن موجوداً.

## محتوى ملف التكوين
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
يدعم `stores.driver` أربعة محركات: **file** و **redis** و **array** و **apcu**.

#### محرك file
المحرك الافتراضي. لا يعتمد على مكونات خارجية. يدعم مشاركة التخزين المؤقت عبر العمليات. لا يدعم المشاركة عبر عدة خوادم.

#### محرك array
تخزين في الذاكرة بأفضل أداء، لكنه يستهلك الذاكرة. لا يدعم المشاركة بين العمليات أو الخوادم. تضيع البيانات عند إعادة تشغيل العملية. مناسب للمشاريع ذات حجم تخزين مؤقت صغير.

#### محرك apcu
تخزين في الذاكرة. أداؤه ثاني أفضل بعد array. يدعم مشاركة التخزين المؤقت عبر العمليات. لا يدعم المشاركة عبر عدة خوادم. تضيع البيانات عند إعادة تشغيل العملية. مناسب للمشاريع ذات حجم تخزين مؤقت صغير.

> يتطلب تثبيت وتفعيل [إضافة APCu](https://pecl.php.net/package/APCu). لا يُنصح به لسيناريوهات الكتابة/الحذف المتكررة للتخزين المؤقت، فقد يؤدي إلى انخفاض ملحوظ في الأداء.

#### محرك redis
يعتمد على مكون [webman/redis](./redis.md). يدعم مشاركة التخزين المؤقت عبر العمليات والخوادم.

**stores.redis.connection**

`stores.redis.connection` يقابل المفتاح المُعرّف في `config/redis.php`. عند استخدام Redis، يعيد استخدام تكوين `webman/redis` بما فيه إعدادات تجمع الاتصالات.

**يُوصى بإضافة تكوين Redis مخصص للتخزين المؤقت في `config/redis.php`، مثال:**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== إضافة جديد
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

ثم ضع `stores.redis.connection` على `cache`. يكون ملف `config/cache.php` النهائي كالتالي:

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

## تبديل التخزين
يمكن التبديل يدوياً بين التخزينات لاستخدام محركات مختلفة، مثال:

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **تلميح**
> أسماء مفاتيح التخزين المؤقت مقيدة بـ [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) ولا يجوز أن تحتوي على أي من الأحرف `{}()/\@:`. اعتباراً من `symfony/cache` 7.2.4، يمكن تجاوز هذا التحقق عبر خيار PHP ini `zend.assertions=-1`.

## استخدام مكونات تخزين مؤقت أخرى

راجع [قواعد البيانات الأخرى](others.md#ThinkCache) لمكون [ThinkCache](https://github.com/webman-php/think-cache).
