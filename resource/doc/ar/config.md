# ملفات التكوين

## الموقع
توجد ملفات تكوين webman في مجلد `config/`. يمكنك استخدام الدالة `config()` للوصول إلى التكوينات المقابلة في مشروعك.

## الوصول إلى التكوينات

الحصول على جميع التكوينات:
```php
config();
```

الحصول على جميع التكوينات في `config/app.php`:
```php
config('app');
```

الحصول على تكوين `debug` في `config/app.php`:
```php
config('app.debug');
```

إذا كان التكوين مصفوفة، يمكنك استخدام `.` للوصول إلى القيم المتداخلة. مثال:
```php
config('file.key1.key2');
```

## القيمة الافتراضية
```php
config($key, $default);
```
مرر القيمة الافتراضية كمعامل ثانٍ. إذا لم يكن التكوين موجوداً، سيتم إرجاع القيمة الافتراضية. إذا لم يكن التكوين موجوداً ولم يتم تعيين قيمة افتراضية، يتم إرجاع `null`.

## التكوين المخصص
يمكن للمطورين إضافة ملفات تكوين خاصة بهم في مجلد `config/`. مثال:

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**الاستخدام عند الوصول إلى التكوينات**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## تعديل التكوينات
لا يدعم Webman تغييرات التكوين الديناميكية. يجب تعديل جميع التكوينات يدوياً في ملفات التكوين المقابلة، ثم إعادة تحميل أو إعادة تشغيل التطبيق.

> **ملاحظة**
> تكوين الخادم `config/server.php` وتكوين العملية `config/process.php` لا يدعمان إعادة التحميل. يجب إعادة التشغيل لتفعيل التغييرات.

## تذكير مهم
إذا قمت بإنشاء ملفات تكوين في مجلد فرعي تحت `config/`، على سبيل المثال `config/order/status.php`، فستحتاج إلى ملف `app.php` في مجلد `config/order/` بالمحتوى التالي:
```php
<?php
return [
    'enable' => true,
];
```
تعيين `enable` على `true` يخبر الإطار بتحميل التكوينات من هذا المجلد.
يجب أن تبدو بنية مجلد التكوين كما يلي:
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
يمكنك بعد ذلك الوصول إلى المصفوفة أو المفاتيح المحددة من `status.php` عبر `config.order.status`.


## مرجع ملفات التكوين

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // منفذ الاستماع (تمت إزالته منذ 1.6.0، يُكوّن في config/process.php)
    'transport' => 'tcp', // بروتوكول النقل (تمت إزالته منذ 1.6.0، يُكوّن في config/process.php)
    'context' => [], // SSL إلخ (تمت إزالته منذ 1.6.0، يُكوّن في config/process.php)
    'name' => 'webman', // اسم العملية (تمت إزالته منذ 1.6.0، يُكوّن في config/process.php)
    'count' => cpu_count() * 4, // عدد العمليات (تمت إزالته منذ 1.6.0، يُكوّن في config/process.php)
    'user' => '', // المستخدم (تمت إزالته منذ 1.6.0، يُكوّن في config/process.php)
    'group' => '', // المجموعة (تمت إزالته منذ 1.6.0، يُكوّن في config/process.php)
    'reusePort' => false, // تفعيل إعادة استخدام المنفذ (تمت إزالته منذ 1.6.0، يُكوّن في config/process.php)
    'event_loop' => '',  // فئة حلقة الأحداث، تُختار تلقائياً افتراضياً
    'stop_timeout' => 2, // أقصى وقت انتظار عند استلام إشارة الإيقاف/إعادة التشغيل/إعادة التحميل، إنهاء قسري إذا لم تخرج العملية في الوقت
    'pid_file' => runtime_path() . '/webman.pid', // موقع ملف PID
    'status_file' => runtime_path() . '/webman.status', // موقع ملف الحالة
    'stdout_file' => runtime_path() . '/logs/stdout.log', // موقع ملف stdout، يُكتب كل المخرجات بعد بدء webman هنا
    'log_file' => runtime_path() . '/logs/workerman.log', // موقع ملف سجل Workerman
    'max_package_size' => 10 * 1024 * 1024 // الحد الأقصى لحجم الحزمة، 10 ميجا. حجم ملف الرف محدود بهذا
];
```

#### app.php
```php
return [
    'debug' => true,  // وضع التصحيح، يُفعّل تتبع المكدس إلخ عند الأخطاء. يُفترض تعطيله في بيئة الإنتاج
    'error_reporting' => E_ALL, // مستوى الإبلاغ عن الأخطاء
    'default_timezone' => 'Asia/Shanghai', // المنطقة الزمنية الافتراضية
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // مسار المجلد العام
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // مسار مجلد التشغيل
    'controller_suffix' => 'Controller', // لاحقة المتحكم
    'controller_reuse' => false, // إعادة استخدام المتحكمات أم لا
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // تكوين عملية webman
    'webman' => [ 
        'handler' => Http::class, // فئة معالج العملية
        'listen' => 'http://0.0.0.0:8787', // عنوان الاستماع
        'count' => cpu_count() * 4, // عدد العمليات، 4x CPU افتراضياً
        'user' => '', // مستخدم العملية، استخدم مستخدماً بصلاحيات منخفضة
        'group' => '', // مجموعة العملية، استخدم مجموعة بصلاحيات منخفضة
        'reusePort' => false, // تفعيل reusePort، يوزع الاتصالات على عمليات worker
        'eventLoop' => '', // فئة حلقة الأحداث، تستخدم server.event_loop عند الفراغ
        'context' => [], // سياق الاستماع، مثلاً SSL
        'constructor' => [ // معاملات المُنشئ لمعالج العملية، هنا فئة Http
            'requestClass' => Request::class, // فئة الطلب المخصصة
            'logger' => Log::channel('default'), // مثيل المسجل
            'appPath' => app_path(), // مسار مجلد التطبيق
            'publicPath' => public_path() // مسار المجلد العام
        ]
    ],
    // عملية المراقبة لإعادة التحميل التلقائي عند تغيير الملفات واكتشاف تسرب الذاكرة
    'monitor' => [
        'handler' => app\process\Monitor::class, // فئة المعالج
        'reloadable' => false, // هذه العملية لا تنفذ إعادة التحمل
        'constructor' => [ // معاملات المُنشئ لمعالج العملية
            // المجلدات للمراقبة، تجنب الإكثار لأن ذلك يبطئ الاكتشاف
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // امتدادات الملفات لمراقبة التغييرات
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // خيارات أخرى
            'options' => [
                // تفعيل مراقبة الملفات، Linux فقط، مراقبة الملفات معطلة افتراضياً في وضع daemon
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // تفعيل مراقبة الذاكرة، Linux فقط
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// إرجاع مثيل حاوية حقن التبعيات PSR-11
return new Webman\Container;
```

#### dependence.php
```php
// تكوين الخدمات والتبعيات في حاوية حقن التبعيات
return [];
```

#### route.php
```php

use support\Route;
// تعريف المسار لـ /test
Route::any('/test', function (Request $request) {
    return response('test');
});
```

#### view.php
```php
use support\view\Raw;
use support\view\Twig;
use support\view\Blade;
use support\view\ThinkPHP;

return [
    'handler' => Raw::class // فئة معالج العرض الافتراضية
];
```

### autoload.php
```php
// تكوين ملفات التحميل التلقائي للإطار
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php',
        base_path() . '/support/Response.php',
    ]
];
```

#### cache.php
```php
// تكوين التخزين المؤقت
return [
    'default' => 'file', // السائق الافتراضي
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // مسار تخزين ملفات التخزين المؤقت
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // اسم اتصال Redis، يشير إلى التكوين في redis.php
        ],
        'array' => [
            'driver' => 'array' // تخزين مؤقت في الذاكرة، يُمسح عند إعادة التشغيل
        ]
    ]
];
```

#### redis.php
```php
return [
    'default' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 0,
    ],
];
```

#### database.php
```php
return [
 // قاعدة البيانات الافتراضية
 'default' => 'mysql',
 // تكوينات اتصالات قاعدة البيانات
 'connections' => [

     'mysql' => [
         'driver'      => 'mysql',
         'host'        => '127.0.0.1',
         'port'        => 3306,
         'database'    => 'webman',
         'username'    => 'webman',
         'password'    => '',
         'unix_socket' => '',
         'charset'     => 'utf8',
         'collation'   => 'utf8_unicode_ci',
         'prefix'      => '',
         'strict'      => true,
         'engine'      => null,
     ],

     'sqlite' => [
         'driver'   => 'sqlite',
         'database' => '',
         'prefix'   => '',
     ],

     'pgsql' => [
         'driver'   => 'pgsql',
         'host'     => '127.0.0.1',
         'port'     => 5432,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
         'schema'   => 'public',
         'sslmode'  => 'prefer',
     ],

     'sqlsrv' => [
         'driver'   => 'sqlsrv',
         'host'     => 'localhost',
         'port'     => 1433,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
     ],
 ],
];
```

#### exception.php
```php
return [
    // تعيين فئة معالج الاستثناءات
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // المعالج
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // اسم ملف السجل
                    7, //$maxFiles // الاحتفاظ بالسجلات لمدة 7 أيام
                    Monolog\Logger::DEBUG, // مستوى السجل
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // المُنسِّق
                    'constructor' => [null, 'Y-m-d H:i:s', true], // معاملات المُنسِّق
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // النوع
    'type' => 'file', // أو redis أو redis_cluster
     // المعالج
    'handler' => FileSessionHandler::class,
     // التكوين
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // مجلد التخزين
        ],
        'redis' => [
            'host' => '127.0.0.1',
            'port' => 6379,
            'auth' => '',
            'timeout' => 2,
            'database' => '',
            'prefix' => 'redis_session_',
        ],
        'redis_cluster' => [
            'host' => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth' => '',
            'prefix' => 'redis_session_',
        ]
    ],
    'session_name' => 'PHPSID', // اسم الجلسة
    'auto_update_timestamp' => false, // تحديث الطابع الزمني تلقائياً لمنع انتهاء صلاحية الجلسة
    'lifetime' => 7*24*60*60, // المدة
    'cookie_lifetime' => 365*24*60*60, // مدة ملف تعريف الارتباط
    'cookie_path' => '/', // مسار ملف تعريف الارتباط
    'domain' => '', // نطاق ملف تعريف الارتباط
    'http_only' => true, // HTTP فقط
    'secure' => false, // HTTPS فقط
    'same_site' => '', // خاصية SameSite
    'gc_probability' => [1, 1000], // احتمال جمع نفايات الجلسة
];
```

#### middleware.php
```php
// تكوين البرمجيات الوسيطة
return [];
```

#### static.php
```php
return [
    'enable' => true, // تفعيل تقديم الملفات الثابتة لـ webman
    'middleware' => [ // برمجية وسيطة للملفات الثابتة، لسياسة التخزين المؤقت، CORS إلخ
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // اللغة الافتراضية
    'locale' => 'zh_CN',
    // اللغات الاحتياطية
    'fallback_locale' => ['zh_CN', 'en'],
    // موقع ملف الترجمة
    'path' => base_path() . '/resource/translations',
];
```
