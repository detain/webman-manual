# ไฟล์การตั้งค่า

## ตำแหน่ง
ไฟล์การตั้งค่าของ webman อยู่ในไดเรกทอรี `config/` และสามารถใช้ฟังก์ชัน `config()` ในโปรเจคเพื่อดึงค่าการตั้งค่าที่เกี่ยวข้อง

## การดึงค่าการตั้งค่า

ดึงค่าการตั้งค่าทั้งหมด
```php
config();
```

ดึงค่าการตั้งค่าทั้งหมดใน `config/app.php`
```php
config('app');
```

ดึงค่า `debug` ใน `config/app.php`
```php
config('app.debug');
```

หากการตั้งค่าเป็นอาร์เรย์ สามารถใช้ `.` เพื่อดึงค่าภายในอาร์เรย์ ตัวอย่างเช่น
```php
config('file.key1.key2');
```

## ค่าเริ่มต้น
```php
config($key, $default);
```
config สามารถใส่ค่าเริ่มต้นผ่านพารามิเตอร์ที่สอง หากไม่มีการตั้งค่าจะคืนค่าเริ่มต้น
หากไม่มีการตั้งค่าและไม่ได้ตั้งค่าเริ่มต้น จะคืนค่าเป็น null

## การตั้งค่าเอง
นักพัฒนาสามารถเพิ่มไฟล์การตั้งค่าของตัวเองในไดเรกทอรี `config/` เช่น

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**การใช้เมื่อดึงค่า**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## เปลี่ยนค่าการตั้งค่า
webman ไม่รองรับการเปลี่ยนค่าการตั้งค่าแบบ dynamic การตั้งค่าทั้งหมดจะต้องเปลี่ยนด้วยตนเองในไฟล์การตั้งค่าที่เกี่ยวข้อง และรีโหลดหรือรีสตาร์ททำให้การเปลี่ยนแปลงมีผล

> **ข้อควรระวัง**
> การตั้งค่าเซิร์ฟเวอร์ใน `config/server.php` และการตั้งค่ากระบวนการใน `config/process.php` ไม่รองรับการรีโหลด จำเป็นต้องรีสตาร์ทเพื่อให้การเปลี่ยนแปลงมีผลลัพธ์

## ข้อควรจำเป็นพิเศษ
หากคุณจะสร้างไฟล์การตั้งค่าในไดเรกทอรีย่อยภายใต้ `config/` เช่น `config/order/status.php` โฟลเดอร์ `config/order` จะต้องมีไฟล์ `app.php` โดยมีเนื้อหาดังนี้
```php
<?php
return [
    'enable' => true,
];
```
`enable` เป็น `true` หมายถึงให้เฟรมเวิร์กอ่านการตั้งค่าจากไดเรกทอรีนี้
โครงสร้างโฟลเดอร์การตั้งค่าสุดท้ายจะคล้ายกับแบบนี้
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
จากนั้นคุณสามารถอ่านอาร์เรย์หรือข้อมูล key เฉพาะจาก `status.php` ผ่าน `config.order.status` ได้


## อธิบายไฟล์การตั้งค่า

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // พอร์ตฟัง (ลบออกตั้งแต่ 1.6.0, ตั้งค่าใน config/process.php)
    'transport' => 'tcp', // โปรโตคอลการขนส่ง (ลบออกตั้งแต่ 1.6.0, ตั้งค่าใน config/process.php)
    'context' => [], // ssl เป็นต้น (ลบออกตั้งแต่ 1.6.0, ตั้งค่าใน config/process.php)
    'name' => 'webman', // ชื่อกระบวนการ (ลบออกตั้งแต่ 1.6.0, ตั้งค่าใน config/process.php)
    'count' => cpu_count() * 4, // จำนวนกระบวนการ (ลบออกตั้งแต่ 1.6.0, ตั้งค่าใน config/process.php)
    'user' => '', // ผู้ใช้ (ลบออกตั้งแต่ 1.6.0, ตั้งค่าใน config/process.php)
    'group' => '', // กลุ่มผู้ใช้ (ลบออกตั้งแต่ 1.6.0, ตั้งค่าใน config/process.php)
    'reusePort' => false, // เปิดใช้พอร์ตซ้ำ (ลบออกตั้งแต่ 1.6.0, ตั้งค่าใน config/process.php)
    'event_loop' => '',  // คลาส event loop ค่าเริ่มต้นเลือกอัตโนมัติ
    'stop_timeout' => 2, // เวลารอสูงสุดเมื่อรับสัญญาณ stop/restart/reload บังคับออกหากกระบวนการไม่สิ้นสุดทันเวลา
    'pid_file' => runtime_path() . '/webman.pid', // ตำแหน่งไฟล์ pid
    'status_file' => runtime_path() . '/webman.status', // ตำแหน่งไฟล์ status
    'stdout_file' => runtime_path() . '/logs/stdout.log', // ตำแหน่งไฟล์ stdout ผลลัพธ์ทั้งหมดหลัง webman เริ่มทำงานจะเขียนที่นี่
    'log_file' => runtime_path() . '/logs/workerman.log', // ตำแหน่งไฟล์ log workerman
    'max_package_size' => 10 * 1024 * 1024 // ขนาดแพ็กเก็ตสูงสุด 10M ขนาดไฟล์อัปโหลดถูกจำกัดโดยค่านี้
];
```

#### app.php
```php
return [
    'debug' => true,  // โหมด debug เปิดแสดง stack trace เมื่อเกิดข้อผิดพลาด ควรปิดในสภาพแวดล้อมผลิต
    'error_reporting' => E_ALL, // ระดับการรายงานข้อผิดพลาด
    'default_timezone' => 'Asia/Shanghai', // โซนเวลาเริ่มต้น
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // ตำแหน่งโฟลเดอร์ public
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // ตำแหน่งโฟลเดอร์ runtime
    'controller_suffix' => 'Controller', // ปัจจัยต่อท้าย controller
    'controller_reuse' => false, // ใช้ controller ซ้ำหรือไม่
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // การตั้งค่ากระบวนการ webman
    'webman' => [ 
        'handler' => Http::class, // คลาสตัวจัดการกระบวนการ
        'listen' => 'http://0.0.0.0:8787', // ที่อยู่ฟัง
        'count' => cpu_count() * 4, // จำนวนกระบวนการ ค่าเริ่มต้น 4 เท่าของ cpu
        'user' => '', // ผู้ใช้กระบวนการ ควรใช้ผู้ใช้สิทธิ์ต่ำ
        'group' => '', // กลุ่มกระบวนการ ควรใช้กลุ่มสิทธิ์ต่ำ
        'reusePort' => false, // เปิด reusePort แจกจ่ายการเชื่อมต่อไปยัง worker
        'eventLoop' => '', // คลาส event loop ใช้ server.event_loop เมื่อว่าง
        'context' => [], // บริบทการฟัง เช่น ssl
        'constructor' => [ // พารามิเตอร์ constructor ของตัวจัดการ ที่นี่เป็นคลาส Http
            'requestClass' => Request::class, // คลาส request ที่กำหนดเอง
            'logger' => Log::channel('default'), // อินสแตนซ์ logger
            'appPath' => app_path(), // ตำแหน่งโฟลเดอร์ app
            'publicPath' => public_path() // ตำแหน่งโฟลเดอร์ public
        ]
    ],
    // กระบวนการ monitor สำหรับตรวจจับการเปลี่ยนแปลงไฟล์และ memory leak
    'monitor' => [
        'handler' => app\process\Monitor::class, // คลาสตัวจัดการ
        'reloadable' => false, // กระบวนการนี้ไม่ดำเนินการ reload
        'constructor' => [ // พารามิเตอร์ constructor ของตัวจัดการ
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
return new Webman\Container;
```

#### dependence.php
```php
return [];
```

#### route.php
```php
use support\Route;
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
    'handler' => Raw::class
];
```

### autoload.php
```php
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
 'default' => 'mysql',
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
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class,
                'constructor' => [
                    runtime_path() . '/logs/webman.log',
                    7,
                    Monolog\Logger::DEBUG,
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class,
                    'constructor' => [null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
    'type' => 'file',
    'handler' => FileSessionHandler::class,
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions',
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
    'session_name' => 'PHPSID',
    'auto_update_timestamp' => false,
    'lifetime' => 7*24*60*60,
    'cookie_lifetime' => 365*24*60*60,
    'cookie_path' => '/',
    'domain' => '',
    'http_only' => true,
    'secure' => false,
    'same_site' => '',
    'gc_probability' => [1, 1000],
];
```

#### middleware.php
```php
return [];
```

#### static.php
```php
return [
    'enable' => true,
    'middleware' => [],
];
```

#### translation.php
```php
return [
    'locale' => 'zh_CN',
    'fallback_locale' => ['zh_CN', 'en'],
    'path' => base_path() . '/resource/translations',
];
```
