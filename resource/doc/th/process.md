# กระบวนการที่กำหนดเอง

ใน webman คุณสามารถกำหนด listener หรือกระบวนการได้เช่นเดียวกับ workerman

> **หมายเหตุ**
> ผู้ใช้ Windows ต้องใช้ `php windows.php` เพื่อเริ่ม webman เพื่อให้สามารถรันกระบวนการที่กำหนดเองได้

## บริการ HTTP ที่กำหนดเอง
บางครั้งคุณอาจมีความต้องการพิเศษที่ต้องแก้ไขโค้ดหลักของบริการ HTTP ใน webman ในกรณีนี้สามารถใช้กระบวนการที่กำหนดเองได้

เช่น สร้างไฟล์ `app\Server.php`

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // เขียนทับเมธอดใน Webman\App ที่นี่
}
```

เพิ่มการตั้งค่าต่อไปนี้ใน `config/process.php`

```php
use Workerman\Worker;

return [
    // ... การตั้งค่าอื่นๆ ข้ามไป ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // จำนวนกระบวนการ
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // ตั้งค่าคลาสคำขอ
            'logger' => \support\Log::channel('default'), // อินสแตนซ์ล็อก
            'appPath' => app_path(), // ตำแหน่งไดเรกทอรี app
            'publicPath' => public_path() // ตำแหน่งไดเรกทอรี public
        ]
    ]
];
```

> **เคล็ดลับ**
> หากต้องการปิดกระบวนการ HTTP ที่มาพร้อม webman เพียงตั้งค่า `listen=>''` ใน config/server.php

## ตัวอย่าง listener WebSocket ที่กำหนดเอง

สร้าง `app/Pusher.php`

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

> หมายเหตุ: เมธอด onXXX ทั้งหมดต้องเป็น public

เพิ่มการตั้งค่าต่อไปนี้ใน `config/process.php`

```php
return [
    // ... การตั้งค่ากระบวนการอื่นๆ ข้ามไป ...
    
    // websocket_test คือชื่อกระบวนการ
    'websocket_test' => [
        // ระบุคลาสกระบวนการที่นี่ คือคลาส Pusher ที่กำหนดไว้ด้านบน
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## ตัวอย่างกระบวนการที่ไม่ฟังที่กำหนดเอง

สร้าง `app/TaskTest.php`

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // ตรวจสอบฐานข้อมูลทุก 10 วินาทีว่ามีผู้ใช้ลงทะเบียนใหม่หรือไม่
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

เพิ่มการตั้งค่าต่อไปนี้ใน `config/process.php`

```php
return [
    // ... การตั้งค่ากระบวนการอื่นๆ ข้ามไป ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> หมายเหตุ: หากละเว้น listen จะไม่ฟังพอร์ตใด หากละเว้น count จำนวนกระบวนการจะถือเป็น 1 โดยค่าเริ่มต้น

## คำอธิบายไฟล์การตั้งค่า

การตั้งค่าที่สมบูรณ์ของกระบวนการกำหนดดังนี้:

```php
return [
    // ... 
    
    // websocket_test คือชื่อกระบวนการ
    'websocket_test' => [
        // ระบุคลาสกระบวนการที่นี่
        'handler' => app\Pusher::class,
        // โปรโตคอล ไอพี และพอร์ตที่ฟัง (ทางเลือก)
        'listen'  => 'websocket://0.0.0.0:8888',
        // จำนวนกระบวนการ (ทางเลือก ค่าเริ่มต้น 1)
        'count'   => 2,
        // ผู้ใช้ที่รันกระบวนการ (ทางเลือก ค่าเริ่มต้นผู้ใช้ปัจจุบัน)
        'user'    => '',
        // กลุ่มผู้ใช้ที่รันกระบวนการ (ทางเลือก ค่าเริ่มต้นกลุ่มปัจจุบัน)
        'group'   => '',
        // กระบวนการปัจจุบันรองรับ reload หรือไม่ (ทางเลือก ค่าเริ่มต้น true)
        'reloadable' => true,
        // เปิดใช้งาน reusePort
        'reusePort'  => true,
        // transport (ทางเลือก ตั้งเป็น ssl เมื่อต้องการ SSL ค่าเริ่มต้น tcp)
        'transport'  => 'tcp',
        // context (ทางเลือก เมื่อ transport เป็น ssl ต้องส่งเส้นทางใบรับรอง)
        'context'    => [], 
        // พารามิเตอร์ตัวสร้างคลาสกระบวนการ (ทางเลือก)
        'constructor' => [],
        // กระบวนการนี้เปิดใช้งานหรือไม่
        'enable' => true
    ],
];
```

## สรุป

กระบวนการที่กำหนดเองใน webman จริงๆ แล้วเป็นการห่อหุ้ม workerman แบบง่าย แยกการตั้งค่าออกจากธุรกิจ และ implement callback `onXXX` ของ workerman ผ่านเมธอดคลาส การใช้งานอื่นเหมือนกับ workerman ทุกประการ
