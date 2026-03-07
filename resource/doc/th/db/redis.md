# Redis

[webman/redis](https://github.com/webman-php/redis) ขยาย [illuminate/redis](https://github.com/illuminate/redis) ด้วยการเพิ่ม connection pool รองรับทั้งสภาพแวดล้อมแบบ coroutine และไม่ใช้ coroutine การใช้งานเหมือนกับ Laravel

ก่อนใช้ `illuminate/redis` ต้องติดตั้ง redis extension สำหรับ `php-cli` ก่อน

## การติดตั้ง

```php
composer require -W webman/redis illuminate/events
```

หลังจากติดตั้งแล้วต้อง restart (reload ไม่ทำงาน)

## การกำหนดค่า

ไฟล์กำหนดค่า Redis อยู่ที่ `config/redis.php`

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // การตั้งค่า connection pool
            'max_connections' => 10,     // จำนวนการเชื่อมต่อสูงสุดใน pool
            'min_connections' => 1,      // จำนวนการเชื่อมต่อขั้นต่ำใน pool
            'wait_timeout' => 3,         // เวลารอสูงสุดเมื่อขอการเชื่อมต่อ (วินาที)
            'idle_timeout' => 50,        // หลังเวลานี้ การเชื่อมต่อจะถูกปล่อยจนถึง min_connections
            'heartbeat_interval' => 50,  // ช่วง heartbeat (ไม่เกิน 60 วินาที)
        ],
    ]
];
```

## เกี่ยวกับ Connection Pool

* แต่ละ process มี pool ของตัวเอง ไม่มีการแชร์ pool ระหว่าง process
* เมื่อไม่ใช้ coroutine การทำงานจะเป็นแบบลำดับ จึงใช้การเชื่อมต่อสูงสุด 1 การเชื่อมต่อ
* เมื่อใช้ coroutine การทำงานจะเป็นแบบขนาน และ pool จะปรับแบบไดนามิกระหว่าง `min_connections` ถึง `max_connections`
* เมื่อจำนวน coroutine ที่ใช้ Redis เกิน `max_connections` จะรอสูงสุด `wait_timeout` วินาที ถ้าเกินจะ throw exception
* เมื่อ idle (มีหรือไม่มี coroutine) การเชื่อมต่อจะถูกปล่อยหลัง `idle_timeout` จนถึง `min_connections` (0 ได้)

## ตัวอย่าง

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

## Redis API

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

เทียบเท่ากับ

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

> **โปรดทราบ**
> ใช้ `Redis::select($db)` อย่างระมัดระวัง เนื่องจาก webman เป็น framework แบบ resident-in-memory การเปลี่ยนฐานข้อมูลในคำขอหนึ่งจะส่งผลต่อคำขอถัดไป สำหรับหลายฐานข้อมูลแนะนำให้กำหนดการเชื่อมต่อ Redis แยกกันสำหรับแต่ละ `$db`

## การใช้การเชื่อมต่อ Redis หลายรายการ

ตัวอย่างในไฟล์กำหนดค่า `config/redis.php`

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

ค่าเริ่มต้นจะใช้การเชื่อมต่อที่กำหนดใน `default` สามารถใช้เมธอด `Redis::connection()` เพื่อเลือกการเชื่อมต่อ Redis ที่ต้องการใช้

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## การกำหนดค่า Cluster

หากแอปพลิเคชันของคุณใช้ Redis server cluster ควรกำหนดค่า cluster ในไฟล์กำหนดค่า Redis ด้วยคีย์ `clusters`

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

โดยค่าเริ่มต้น cluster สามารถทำ client-side sharding บน nodes ได้ ทำให้สามารถสร้าง node pool และใช้งานหน่วยความจำจำนวนมาก โปรดทราบว่า client-side sharding ไม่ได้จัดการกรณีเกิดความล้มเหลว ดังนั้นฟีเจอร์นี้เหมาะสำหรับข้อมูลแคชที่ได้จากฐานข้อมูลหลักอื่น หากต้องการใช้ Redis native cluster ต้องกำหนดค่าใน options ของไฟล์กำหนดค่าดังนี้

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

## คำสั่ง Pipeline

เมื่อต้องการส่งคำสั่งหลายรายการไปยังเซิร์ฟเวอร์ในการดำเนินการครั้งเดียว แนะนำให้ใช้คำสั่ง pipeline เมธอด `pipeline` รับ closure ของ Redis instance คุณสามารถส่งคำสั่งทั้งหมดไปยัง Redis instance ได้ และจะดำเนินการในครั้งเดียว

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
