# العمليات المخصصة

في webman يمكنك تخصيص المستمعين أو العمليات تمامًا كما في workerman.

> **ملاحظة**
> يحتاج مستخدمو Windows إلى استخدام `php windows.php` لتشغيل webman حتى تتمكن من تشغيل العمليات المخصصة.

## خدمة HTTP المخصصة
أحيانًا قد تكون لديك متطلبات خاصة تتطلب تعديل الكود الأساسي لخدمة HTTP الخاصة بـ webman. في هذه الحالة يمكنك استخدام عملية مخصصة لتحقيق ذلك.

على سبيل المثال، أنشئ ملف `app\Server.php`.

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // أعد كتابة طرق Webman\App هنا
}
```

أضف التكوين التالي إلى `config/process.php`.

```php
use Workerman\Worker;

return [
    // ... تم حذف التكوينات الأخرى ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // عدد العمليات
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // إعداد فئة الطلب
            'logger' => \support\Log::channel('default'), // مثيل السجل
            'appPath' => app_path(), // موقع مجلد app
            'publicPath' => public_path() // موقع مجلد public
        ]
    ]
];
```

> **تلميح**
> لإيقاف عملية HTTP المدمجة في webman، ما عليك سوى تعيين `listen=>''` في `config/server.php`.

## مثال مستمع WebSocket المخصص

أنشئ `app/Pusher.php`.

```php
<?php
namespace app;

use Workerman\Connection\TcpConnection;

class Pusher
{
    public function onConnect(TcpConnection $connection)
    {
        echo "onConnect\n";
    }

    public function onWebSocketConnect(TcpConnection $connection, $http_buffer)
    {
        echo "onWebSocketConnect\n";
    }

    public function onMessage(TcpConnection $connection, $data)
    {
        $connection->send($data);
    }

    public function onClose(TcpConnection $connection)
    {
        echo "onClose\n";
    }
}
```

> ملاحظة: يجب أن تكون جميع الطرق onXXX عامة (public).

أضف التكوين التالي إلى `config/process.php`.

```php
return [
    // ... تم حذف تكوينات العمليات الأخرى ...
    
    // websocket_test هو اسم العملية
    'websocket_test' => [
        // حدد فئة العملية هنا، أي فئة Pusher المعرفة أعلاه
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## مثال عملية غير مستمعة مخصصة

أنشئ `app/TaskTest.php`.

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // تحقق من قاعدة البيانات كل 10 ثوانٍ لمعرفة ما إذا كان هناك مستخدمون جديدون مسجلون
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

أضف التكوين التالي إلى `config/process.php`.

```php
return [
    // ... تم حذف تكوينات العمليات الأخرى ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> ملاحظة: إذا تم حذف listen فلن تستمع العملية لأي منفذ. وإذا تم حذف count فسيكون العدد الافتراضي للعمليات 1.

## شرح ملف التكوين

يُعرِّف تكوين العملية الكامل على النحو التالي:

```php
return [
    // ... 
    
    // websocket_test هو اسم العملية
    'websocket_test' => [
        // حدد فئة العملية هنا
        'handler' => app\Pusher::class,
        // البروتوكول وعنوان IP والمنفذ للاستماع (اختياري)
        'listen'  => 'websocket://0.0.0.0:8888',
        // عدد العمليات (اختياري، الافتراضي 1)
        'count'   => 2,
        // المستخدم لتشغيل العملية (اختياري، الافتراضي المستخدم الحالي)
        'user'    => '',
        // مجموعة المستخدمين لتشغيل العملية (اختياري، الافتراضي المجموعة الحالية)
        'group'   => '',
        // هل تدعم العملية الحالية إعادة التحميل (اختياري، الافتراضي true)
        'reloadable' => true,
        // تفعيل reusePort
        'reusePort'  => true,
        // النقل (اختياري، اضبط على 'ssl' عند الحاجة إلى SSL، الافتراضي 'tcp')
        'transport'  => 'tcp',
        // السياق (اختياري، مرر مسار الشهادة عندما يكون النقل 'ssl')
        'context'    => [], 
        // معلمات منشئ فئة العملية (اختياري)
        'constructor' => [],
        // هل هذه العملية مفعّلة
        'enable' => true
    ],
];
```

## الخلاصة

العمليات المخصصة في webman هي في الأساس تغليف بسيط لـ workerman. تفصل التكوين عن منطق الأعمال وتنفذ استدعاءات `onXXX` الخاصة بـ workerman من خلال طرق الفئة. الاستخدام الآخر مطابق تمامًا لـ workerman.
