# قائمة انتظار Redis

قائمة انتظار رسائل مبنية على Redis، تدعم معالجة الرسائل المؤجلة.

## التثبيت
`composer require webman/redis-queue`

## ملف التكوين
يتم إنشاء ملف تكوين Redis تلقائياً في `{المشروع-الرئيسي}/config/plugin/webman/redis-queue/redis.php`، ومحتواه مشابه لما يلي:
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // كلمة المرور، اختياري
            'db' => 0,            // قاعدة البيانات
            'max_attempts'  => 5, // عدد إعادة المحاولة بعد فشل الاستهلاك
            'retry_seconds' => 5, // الفاصل الزمني لإعادة المحاولة بالثواني
        ]
    ],
];
```

### إعادة المحاولة عند فشل الاستهلاك
إذا فشل الاستهلاك (حدث استثناء)، ستُوضع الرسالة في قائمة الانتظار المؤجلة وستنتظر المحاولة التالية. يتم التحكم في عدد إعادة المحاولة بواسطة `max_attempts`، والفاصل الزمني يُتحكم به مشتركاً من `retry_seconds` و`max_attempts`. مثلاً إذا كان `max_attempts` يساوي 5 و`retry_seconds` يساوي 10، ففاصل إعادة المحاولة الأولى هو `1*10` ثانية، والثانية `2*10` ثانية، والثالثة `3*10` ثانية، وهكذا حتى 5 محاولات. إذا تجاوز عدد إعادة المحاولة إعداد `max_attempts`، ستُوضع الرسالة في قائمة الفشل بالمفتاح `{redis-queue}-failed`.

## إرسال الرسائل (متزامن)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // اسم القائمة
        $queue = 'send-mail';
        // البيانات، يمكن تمرير مصفوفة مباشرة بدون تسلسل
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // إرسال الرسالة
        Redis::send($queue, $data);
        // إرسال رسالة مؤجلة، تُعالج بعد 60 ثانية
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
عند النجاح يُرجع `Redis::send()` true، وإلا يُرجع false أو يرمي استثناءً.

> **تلميح**
> قد يكون هناك انحراف في وقت استهلاك قائمة الانتظار المؤجلة. مثلاً عندما تكون سرعة الاستهلاك أبطأ من سرعة الإنتاج، قد يسبب تراكم القائمة وتأخر الاستهلاك. الحل هو تشغيل المزيد من عمليات الاستهلاك.

## إرسال الرسائل (غير متزامن)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // اسم القائمة
        $queue = 'send-mail';
        // البيانات، يمكن تمرير مصفوفة مباشرة بدون تسلسل
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // إرسال الرسالة
        Client::send($queue, $data);
        // إرسال رسالة مؤجلة، تُعالج بعد 60 ثانية
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` لا يُرجع قيمة. إنه إرسال غير متزامن ولا يضمن وصول الرسائل 100٪ إلى Redis.

> **تلميح**
> مبدأ `Client::send()` هو إنشاء قائمة انتظار في الذاكرة المحلية ومزامنة الرسائل بشكل غير متزامن مع Redis (المزامنة سريعة، حوالي 10000 رسالة في الثانية). إذا أعيد تشغيل العملية ولم تُكمّل مزامنة البيانات في قائمة الذاكرة المحلية، قد يحدث فقدان رسائل. إرسال `Client::send()` غير المتزامن مناسب للرسائل غير الحرجة.

> **تلميح**
> `Client::send()` غير متزامن ولا يمكن استخدامه إلا في بيئة تشغيل Workerman. لسكريبتات سطر الأوامر استخدم الواجهة المتزامنة `Redis::send()`.

## إرسال الرسائل من مشاريع أخرى
أحياناً تحتاج لإرسال رسائل من مشاريع أخرى ولا يمكنك استخدام `webman\redis-queue`. في هذه الحالات يمكنك الرجوع للدالة التالية لإرسال الرسائل للقائمة.

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

هنا المعامل `$redis` هو مثيل Redis. مثلاً استخدام امتداد redis شبيه بـ:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## الاستهلاك
ملف تكوين عملية الاستهلاك في `{المشروع-الرئيسي}/config/plugin/webman/redis-queue/process.php`. مجلد المستهلك تحت `{المشروع-الرئيسي}/app/queue/redis/`.

تنفيذ الأمر `php webman redis-queue:consumer my-send-mail` سيُنشئ الملف `{المشروع-الرئيسي}/app/queue/redis/MyMailSend.php`.

> **تلميح**
> يتطلب هذا الأمر تثبيت إضافة [الأوامر](../plugin/console.md). إذا لم ترغب بالتثبيت يمكنك إنشاء ملف يدوياً مشابه لما يلي:

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // اسم القائمة للاستهلاك
    public $queue = 'send-mail';

    // اسم الاتصال، مطابق للاتصال في plugin/webman/redis-queue/redis.php
    public $connection = 'default';

    // الاستهلاك
    public function consume($data)
    {
        // لا حاجة لإلغاء التسلسل
        var_export($data); // يطبع ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // رد استدعاء فشل الاستهلاك
    /* 
    $package = [
        'id' => 1357277951, // معرف الرسالة
        'time' => 1709170510, // وقت الرسالة
        'delay' => 0, // وقت التأخير
        'attempts' => 2, // عدد مرات الاستهلاك
        'queue' => 'send-mail', // اسم القائمة
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // محتوى الرسالة
        'max_attempts' => 5, // الحد الأقصى لإعادة المحاولة
        'error' => 'رسالة الخطأ' // رسالة الخطأ
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // لا حاجة لإلغاء التسلسل
        var_export($package); 
    }
}
```

> **ملاحظة**
> يُعتبر الاستهلاك ناجحاً عندما لا يُرمى أي استثناء أو Error أثناء الاستهلاك؛ وإلا فهو فشل وستدخل الرسالة قائمة إعادة المحاولة. redis-queue لا يملك آلية ack؛ يمكن اعتباره ack تلقائي (عند عدم حدوث استثناء أو Error). لتعليم الرسالة الحالية بأنها لم تُستهلك بنجاح، ارمِ استثناءً يدوياً لإرسال الرسالة لقائمة إعادة المحاولة. عملياً هذا لا يختلف عن آلية ack.

> **تلميح**
> المستهلكون يدعمون خوادم وعمليات متعددة، والرسالة نفسها **لن** تُستهلك مرتين. الرسائل المستهلكة تُحذف تلقائياً من القائمة؛ لا حاجة للحذف اليدوي.

> **تلميح**
> عمليات الاستهلاك يمكنها استهلاك قوائم متعددة مختلفة في وقت واحد. إضافة قائمة جديدة لا تتطلب تغيير التكوين في `process.php`. عند إضافة مستهلك قائمة جديد، يُضاف صنف `Consumer` المقابل تحت `app/queue/redis` فحسب، ويُستخدم الخاصية `$queue` لتحديد اسم القائمة للاستهلاك.

> **تلميح**
> مستخدمو Windows يحتاجون لتشغيل `php windows.php` لبدء webman، وإلا لن تبدأ عملية الاستهلاك.

> **تلميح**
> يُستدعى رد استدعاء onConsumeFailure في كل مرة يفشل فيها الاستهلاك. يمكنك معالجة منطق ما بعد الفشل هنا. (هذه الميزة تتطلب `webman/redis-queue>=1.3.2` و`workerman/redis-queue>=1.2.1`)

## ضبط عمليات استهلاك مختلفة لقوائم مختلفة
افتراضياً، كل المستهلكين يشتركون بنفس عملية الاستهلاك. لكن أحياناً نحتاج عزل استهلاك بعض القوائم—مثلاً وضع الأعمال بطيئة الاستهلاك في مجموعة عمليات والأعمال سريعة الاستهلاك في مجموعة أخرى. للقيام بذلك، يمكننا تقسيم المستهلكين إلى مجلدين، مثلاً `app_path() . '/queue/redis/fast'` و`app_path() . '/queue/redis/slow'` (لاحظ أنه يجب تحديث نطاق صنف المستهلك وفقاً لذلك). التكوين كالتالي:
```php
return [
    ...تكوينات أخرى محذوفة...
    
    'redis_consumer_fast'  => [ // المفتاح مخصص، لا قيد على الشكل، هنا سمي redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // مجلد أصناف المستهلك
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // المفتاح مخصص، لا قيد على الشكل، هنا سمي redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // مجلد أصناف المستهلك
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

بهذا الشكل، مستهلكو الأعمال السريعة يذهبون لمجلد `queue/redis/fast` ومستهلكو الأعمال البطيئة لمجلد `queue/redis/slow`، محققين الهدف من تعيين عمليات استهلاك للقوائم.

## تكوين Redis متعدد
#### التكوين
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // كلمة المرور، نوع سلسلة، اختياري
            'db' => 0,           // قاعدة البيانات
            'max_attempts'  => 5, // عدد إعادة المحاولة بعد فشل الاستهلاك
            'retry_seconds' => 5, // الفاصل الزمني لإعادة المحاولة بالثواني
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // كلمة المرور، نوع سلسلة، اختياري
            'db' => 0,            // قاعدة البيانات
            'max_attempts'  => 5, // عدد إعادة المحاولة بعد فشل الاستهلاك
            'retry_seconds' => 5, // الفاصل الزمني لإعادة المحاولة بالثواني
        ]
    ],
];
```

لاحظ أنه تمت إضافة تكوين Redis إضافي بالمفتاح `other`.

#### إرسال الرسائل إلى Redis متعدد

```php
// إرسال رسالة للقائمة بالمفتاح `default`
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// نفس
Client::send($queue, $data);
Redis::send($queue, $data);

// إرسال رسالة للقائمة بالمفتاح `other`
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### الاستهلاك من Redis متعدد
استهلاك الرسائل من القائمة بالمفتاح `other` في التكوين:
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // اسم القائمة للاستهلاك
    public $queue = 'send-mail';

    // === ضبط 'other' هنا للاستهلاك من القائمة بالمفتاح 'other' في التكوين ===
    public $connection = 'other';

    // الاستهلاك
    public function consume($data)
    {
        // لا حاجة لإلغاء التسلسل
        var_export($data);
    }
}
```

## الأسئلة الشائعة

**لماذا يحدث الخطأ `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)`؟**

يحدث هذا الخطأ فقط مع واجهة الإرسال غير المتزامن `Client::send()`. الإرسال غير المتزامن يحفظ الرسائل أولاً في الذاكرة المحلية ثم يرسلها إلى Redis عندما تكون العملية خاملة. إذا كان Redis يستقبل الرسائل أبطأ من إنتاجها، أو كانت العملية مشغولة بمهام أخرى ولا تملك وقتاً كافياً لمزامنة الرسائل من الذاكرة إلى Redis، قد يحدث تراكم رسائل. إذا استمر التراكم أكثر من 600 ثانية، يُثار هذا الخطأ.

الحل: استخدم واجهة الإرسال المتزامن `Redis::send()` لإرسال الرسائل.
