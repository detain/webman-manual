# Redis คิว

คิวข้อความที่ใช้ Redis เป็นฐาน รองรับการประมวลผลข้อความแบบเลื่อนเวลาได้

## การติดตั้ง
`composer require webman/redis-queue`

## ไฟล์การกำหนดค่า
ไฟล์การกำหนดค่า Redis ถูกสร้างอัตโนมัติที่ `{โปรเจกต์หลัก}/config/plugin/webman/redis-queue/redis.php` มีเนื้อหาประมาณนี้:
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // รหัสผ่าน ไม่บังคับ
            'db' => 0,            // ฐานข้อมูล
            'max_attempts'  => 5, // จำนวนครั้งที่ลองใหม่หลัง consume ล้มเหลว
            'retry_seconds' => 5, // ช่วงเวลาระหว่างการลองใหม่ (วินาที)
        ]
    ],
];
```

### การลองใหม่เมื่อ consume ล้มเหลว
ถ้า consume ล้มเหลว (มี exception) ข้อความจะถูกใส่ในคิวเลื่อนเวลาและรอลองใหม่รอบถัดไป จำนวนครั้งที่ลองใหม่ถูกควบคุมโดย `max_attempts` ช่วงเวลาถูกควบคุมร่วมกันโดย `retry_seconds` และ `max_attempts` เช่น `max_attempts` เป็น 5, `retry_seconds` เป็น 10 รอบที่ 1 ห่าง `1*10` วินาที รอบที่ 2 ห่าง `2*10` วินาที รอบที่ 3 ห่าง `3*10` วินาที ไปจนครบ 5 รอบ ถ้าจำนวนครั้งเกินค่าที่ตั้งใน `max_attempts` ข้อความจะถูกใส่ในคิวล้มเหลวที่มีคีย์ `{redis-queue}-failed`

## การส่งข้อความ (ซิงโครนัส)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // ชื่อคิว
        $queue = 'send-mail';
        // ข้อมูล ส่ง array โดยตรงได้ ไม่ต้อง serialize
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // ส่งข้อความ
        Redis::send($queue, $data);
        // ส่งข้อความแบบเลื่อนเวลา จะประมวลผลหลัง 60 วินาที
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
เมื่อส่งสำเร็จ `Redis::send()` คืนค่า true ไม่เช่นนั้น false หรือ throw exception

> **เคล็ดลับ**
> เวลา consume ของคิวเลื่อนเวลาอาจคลาดเคลื่อน เช่น ความเร็ว consume ช้ากว่าผลิต ทำให้คิวสะสมและ consume ล่าช้า วิธีบรรเทา: เปิด process consume เพิ่ม

## การส่งข้อความ (อะซิงโครนัส)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // ชื่อคิว
        $queue = 'send-mail';
        // ข้อมูล ส่ง array โดยตรงได้ ไม่ต้อง serialize
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // ส่งข้อความ
        Client::send($queue, $data);
        // ส่งข้อความแบบเลื่อนเวลา จะประมวลผลหลัง 60 วินาที
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` ไม่คืนค่า เป็นการ push แบบอะซิงโครนัส และไม่รับประกันการส่งถึง Redis 100%

> **เคล็ดลับ**
> หลักการของ `Client::send()` คือสร้างคิวในหน่วยความจำ lokal แล้ว sync ข้อความไป Redis แบบอะซิงโครนัส (sync เร็ว ประมาณ 10,000 ข้อความต่อวินาที) ถ้า process รีสตาร์ทก่อนที่ข้อมูลในคิวหน่วยความจำจะ sync เสร็จ ข้อความอาจสูญหาย `Client::send()` การส่งแบบอะซิงโครนัสเหมาะกับข้อความที่ไม่สำคัญ

> **เคล็ดลับ**
> `Client::send()` เป็นอะซิงโครนัส ใช้ได้เฉพาะในสภาพแวดล้อมรัน Workerman สำหรับสคริปต์บรรทัดคำสั่งให้ใช้ interface ซิงโครนัส `Redis::send()`

## ส่งข้อความจากโปรเจกต์อื่น
บางครั้งต้องส่งข้อความจากโปรเจกต์อื่นและใช้ `webman\redis-queue` ไม่ได้ ในกรณีนี้สามารถใช้ฟังก์ชันต่อไปนี้ส่งข้อความไปยังคิวได้

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

ตรงนี้ พารามิเตอร์ `$redis` คือ Redis instance เช่น การใช้ redis extension คล้ายๆ นี้:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## การ Consume
ไฟล์ config ของ process consumer อยู่ที่ `{โปรเจกต์หลัก}/config/plugin/webman/redis-queue/process.php` ไดเรกทอรี consumer อยู่ที่ `{โปรเจกต์หลัก}/app/queue/redis/`

คำสั่ง `php webman redis-queue:consumer my-send-mail` จะสร้างไฟล์ `{โปรเจกต์หลัก}/app/queue/redis/MyMailSend.php`

> **เคล็ดลับ**
> คำสั่งนี้ต้องติดตั้ง plugin [คอนโซล](../plugin/console.md) ถ้าไม่ต้องการติดตั้งก็สร้างไฟล์เองแบบนี้ได้:

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // ชื่อคิวที่ต้อง consume
    public $queue = 'send-mail';

    // ชื่อการเชื่อมต่อ ตรงกับการเชื่อมต่อใน plugin/webman/redis-queue/redis.php
    public $connection = 'default';

    // การ consume
    public function consume($data)
    {
        // ไม่ต้อง deserialize
        var_export($data); // แสดง ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // callback เมื่อ consume ล้มเหลว
    /* 
    $package = [
        'id' => 1357277951, // message ID
        'time' => 1709170510, // เวลา message
        'delay' => 0, // เวลาเลื่อน
        'attempts' => 2, // จำนวนครั้งที่ consume
        'queue' => 'send-mail', // ชื่อคิว
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // เนื้อหา message
        'max_attempts' => 5, // จำนวนครั้งลองใหม่สูงสุด
        'error' => 'ข้อความ error' // ข้อความ error
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // ไม่ต้อง deserialize
        var_export($package); 
    }
}
```

> **หมายเหตุ**
> ถ้าไม่ throw exception หรือ Error ระหว่าง consume ถือว่าสำเร็จ ไม่เช่นนั้นล้มเหลวและข้อความจะเข้าคิวลองใหม่ redis-queue ไม่มีกลไก ack ดูได้ว่าเป็น auto ack (เมื่อไม่มี exception หรือ Error) ถ้าต้องการบอกว่าข้อความปัจจุบัน consume ไม่สำเร็จ ให้ throw exception ด้วยตัวเองเพื่อให้ข้อความเข้าคิวลองใหม่ จริงๆ แล้วไม่ต่างจากกลไก ack

> **เคล็ดลับ**
> consumer รองรับหลายเซิร์ฟเวอร์หลาย process และข้อความเดียวกันจะ **ไม่** ถูก consume ซ้ำ ข้อความที่ consume แล้วจะถูกลบออกจากคิวอัตโนมัติ ไม่ต้องลบเอง

> **เคล็ดลับ**
> process consumer สามารถ consume หลายคิวต่างกันพร้อมกันได้ การเพิ่มคิวใหม่ไม่ต้องแก้ config ใน `process.php` เมื่อเพิ่ม consumer คิวใหม่ แค่เพิ่มคลาส `Consumer` ที่ตรงกันใต้ `app/queue/redis` และใช้ property `$queue` ระบุชื่อคิวที่ต้อง consume

> **เคล็ดลับ**
> ผู้ใช้ Windows ต้องรัน `php windows.php` เพื่อสตาร์ท webman ไม่เช่นนั้น process consumer จะไม่สตาร์ท

> **เคล็ดลับ**
> callback onConsumeFailure จะถูกเรียกทุกครั้งที่ consume ล้มเหลว สามารถจัดการ logic หลังล้มเหลวที่นี่ได้ (ฟีเจอร์นี้ต้องใช้ `webman/redis-queue>=1.3.2` และ `workerman/redis-queue>=1.2.1`)

## ตั้ง process consumer คนละตัวสำหรับคิวต่างกัน
ตามค่าเริ่มต้น consumer ทั้งหมดใช้ process consume เดียวกัน บางครั้งต้องการแยกการ consume บางคิว—เช่น ธุรกิจที่ consume ช้าใส่ process กลุ่มหนึ่ง ธุรกิจที่ consume เร็วใส่อีกกลุ่ม วิธีทำคือแยก consumer เป็นสองไดเรกทอรี เช่น `app_path() . '/queue/redis/fast'` และ `app_path() . '/queue/redis/slow'` (namespace ของคลาส consumer ต้องแก้ให้ตรงกัน) config แบบนี้:
```php
return [
    ...config อื่น省略...
    
    'redis_consumer_fast'  => [ // key เป็นของเรา ไม่มีข้อจำกัดรูปแบบ ตรงนี้ตั้งเป็น redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // ไดเรกทอรีคลาส consumer
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // key เป็นของเรา ไม่มีข้อจำกัดรูปแบบ ตรงนี้ตั้งเป็น redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // ไดเรกทอรีคลาส consumer
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

แบบนี้ consumer ธุรกิจเร็วไปอยู่ใน `queue/redis/fast` ธุรกิจช้าไปอยู่ใน `queue/redis/slow` เป็นการกำหนด process consume ให้คิวแต่ละตัวได้ตามต้องการ

## การกำหนดค่า Redis หลายตัว
#### การกำหนดค่า
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // รหัสผ่าน ประเภท string ไม่บังคับ
            'db' => 0,            // ฐานข้อมูล
            'max_attempts'  => 5, // จำนวนครั้งลองใหม่หลัง consume ล้มเหลว
            'retry_seconds' => 5, // ช่วงเวลาระหว่างการลองใหม่ (วินาที)
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // รหัสผ่าน ประเภท string ไม่บังคับ
            'db' => 0,            // ฐานข้อมูล
            'max_attempts'  => 5, // จำนวนครั้งลองใหม่หลัง consume ล้มเหลว
            'retry_seconds' => 5, // ช่วงเวลาระหว่างการลองใหม่ (วินาที)
        ]
    ],
];
```

สังเกตว่ามีการเพิ่ม config Redis ที่มี key `other`

#### ส่งข้อความไป Redis หลายตัว

```php
// ส่งข้อความไปคิวที่มี key `default`
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// เหมือนกับ
Client::send($queue, $data);
Redis::send($queue, $data);

// ส่งข้อความไปคิวที่มี key `other`
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### Consume จาก Redis หลายตัว
Consume ข้อความจากคิวที่มี key `other` ใน config:
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // ชื่อคิวที่ต้อง consume
    public $queue = 'send-mail';

    // === ตั้ง 'other' ที่นี่เพื่อ consume จากคิวที่มี key 'other' ใน config ===
    public $connection = 'other';

    // การ consume
    public function consume($data)
    {
        // ไม่ต้อง deserialize
        var_export($data);
    }
}
```

## คำถามที่พบบ่อย

**ทำไมถึง error `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)`**

Error นี้เกิดเฉพาะ interface ส่งแบบอะซิงโครนัส `Client::send()` การส่งแบบอะซิงโครนัสจะเก็บข้อความในหน่วยความจำ lokal ก่อน แล้วค่อยส่งไป Redis เมื่อ process ว่าง ถ้า Redis รับข้อความช้ากว่าที่ผลิต หรือ process ยุ่งกับงานอื่นจนไม่มีเวลา sync ข้อความจากหน่วยความจำไป Redis จะทำให้ข้อความค้าง ถ้าข้อความค้างเกิน 600 วินาที จะ trigger error นี้

วิธีแก้: ใช้ interface ส่งแบบซิงโครนัส `Redis::send()` ในการส่งข้อความ
