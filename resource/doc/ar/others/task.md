# معالجة العمليات البطيئة

أحيانًا نحتاج إلى معالجة عمليات بطيئة لتجنب تأثيرها على معالجة الطلبات الأخرى في webman. يمكن لهذه العمليات استخدام حلول معالجة مختلفة حسب الحالة.

## الحل 1: استخدام قائمة انتظار الرسائل
راجع [قائمة انتظار Redis](../queue/redis.md) [قائمة انتظار Stomp](../queue/stomp.md)

#### المزايا
يمكن التعامل مع طفرات مفاجئة في طلبات المعالجة

#### العيوب
لا يمكن إرجاع النتائج مباشرةً إلى العميل. إذا احتجت لإرسال النتائج، يجب التنسيق مع خدمات أخرى، مثل استخدام [webman/push](https://www.workerman.net/plugin/2) لإرسال نتائج المعالجة.

## الحل 2: إضافة منفذ HTTP جديد

إضافة منفذ HTTP جديد لمعالجة الطلبات البطيئة. يتم معالجة هذه الطلبات البطيئة من خلال مجموعة محددة من العمليات عبر الوصول لهذا المنفذ، ثم إرجاع النتائج مباشرةً إلى العميل بعد المعالجة.

#### المزايا
يمكن إرجاع البيانات مباشرةً إلى العميل

#### العيوب
لا يمكن التعامل مع طفرات مفاجئة في الطلبات

#### خطوات التنفيذ
أضف الإعدادات التالية في `config/process.php`.
```php
return [
    // ... إعدادات أخرى محذوفة ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // عدد العمليات
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // إعداد فئة الطلب
            'logger' => \support\Log::channel('default'), // مثيل المسجل
            'appPath' => app_path(), // موقع مجلد app
            'publicPath' => public_path() // موقع مجلد public
        ]
    ]
];
```

وبهذا يمكن للواجهات البطيئة المرور عبر مجموعة العمليات على `http://127.0.0.1:8686/` دون التأثير على معالجة العمليات الأخرى.

لجعل الواجهة الأمامية لا تلاحظ فرق المنفذ، يمكن إضافة وكيل للمنفذ 8686 في nginx. إذا بدأت مسارات الطلبات البطيئة بـ `/task`، تكون إعدادات nginx مشابهة لما يلي:
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# إضافة upstream جديد 8686
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # الطلبات التي تبدأ بـ /task تذهب للمنفذ 8686، غيّر /task حسب البادئة المطلوبة
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # الطلبات الأخرى تذهب للمنفذ 8787 الأصلي
  location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      if (!-f $request_filename){
          proxy_pass http://webman;
      }
  }
}
```

وبهذا عند وصول العميل لـ `domain.com/task/xxx` سيتم المعالجة عبر المنفذ المنفصل 8686 دون التأثير على معالجة الطلبات على المنفذ 8787.

## الحل 3: استخدام HTTP Chunked لإرسال البيانات المقسمة بشكل غير متزامن

#### المزايا
يمكن إرجاع البيانات مباشرةً إلى العميل

**تثبيت workerman/http-client**

```
composer require workerman/http-client
```

**app/controller/IndexController.php**
```php
<?php
namespace app\controller;

use support\Request;
use support\Response;
use Workerman\Protocols\Http\Chunk;

class IndexController
{
    public function index(Request $request)
    {
        $connection = $request->connection;
        $http = new \Workerman\Http\Client();
        $http->get('https://example.com/', function ($response) use ($connection) {
            $connection->send(new Chunk($response->getBody()));
            $connection->send(new Chunk('')); // إرسال chunk فارغ للدلالة على نهاية الاستجابة
        });
        // إرسال رأس HTTP أولاً، البيانات تُرسل بشكل غير متزامن
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **ملاحظة**
> يستخدم هذا المثال عميل `workerman/http-client` لجلب نتائج HTTP بشكل غير متزامن وإرجاع البيانات. يمكن أيضًا استخدام عملاء غير متزامنين آخرين مثل [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html).
