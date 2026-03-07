# เริ่มต้นด่วนกับฐานข้อมูล (ใช้ Laravel Database Component)

[webman/database](https://github.com/webman-php/database) พัฒนาโดยอิง [illuminate/database](https://github.com/illuminate/database) และเพิ่มฟีเจอร์ connection pool รองรับทั้งสภาพแวดล้อมแบบ coroutine และแบบไม่ใช้ coroutine การใช้งานเหมือนกับ Laravel

ดูได้ที่ [ใช้คอมโพเนนต์ฐานข้อมูลอื่น](others.md) หากต้องการใช้ ThinkPHP หรือฐานข้อมูลอื่น

## ติดตั้งฐานข้อมูล

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

หลังติดตั้งต้อง restart (reload ใช้ไม่ได้)

> **คำแนะนำ**
> webman/database ขึ้นกับ `illuminate/database` ของ Laravel ดังนั้นจะติดตั้งแพ็กเกจ dependency อัตโนมัติ

> **หมายเหตุ**
> ถ้าไม่ต้องการ pagination, database events หรือ SQL logging ให้รันแค่:
> `composer require -W webman/database`

## กำหนดค่าฐานข้อมูล
`config/database.php`
```php

return [
    // ฐานข้อมูลหลัก
    'default' => 'mysql',

    // การตั้งค่าการเชื่อมต่อ
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'test',
            'username'    => 'root',
            'password'    => '',
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
            'options' => [
                PDO::ATTR_EMULATE_PREPARES => false, // จำเป็นเมื่อใช้ swoole หรือ swow
            ],
            'pool' => [ // การตั้งค่า connection pool
                'max_connections' => 5, // จำนวน connection สูงสุด
                'min_connections' => 1, // จำนวน connection ต่ำสุด
                'wait_timeout' => 3,    // เวลารอขอ connection จาก pool สูงสุด เกินแล้วโยน exception ใช้กับ coroutine เท่านั้น
                'idle_timeout' => 60,   // เวลา idle สูงสุดของ connection เกินแล้วปิดเก็บจนเหลือ min_connections
                'heartbeat_interval' => 50, // ช่วงตรวจสอบ heartbeat ของ pool (วินาที) แนะนำน้อยกว่า 60
            ],
        ],
    ],
];
```

นอกจาก `pool` แล้ว ส่วนอื่นเหมือน Laravel

## เกี่ยวกับ Connection Pool
* แต่ละ process มี pool ของตัวเอง ไม่แชร์ระหว่าง processes
* ไม่เปิด coroutine งานใน process จะรันต่อคิว ไม่มี concurrency ดังนั้น pool จะมี connection ไม่เกิน 1 อัน
* เปิด coroutine แล้ว งานจะรันพร้อมกันใน process pool จะปรับจำนวน connection ตามต้องการ ไม่เกิน `max_connections` ไม่ต่ำกว่า `min_connections`
* เพราะ pool สูงสุด `max_connections` ถ้าจำนวน coroutine ที่ใช้ database เกิน จะมี coroutine ต้องรอในคิว สูงสุด `wait_timeout` วินาที เกินแล้วเกิด exception
* เมื่อว่าง (ทั้ง coroutine และไม่ใช้ coroutine) connection จะถูกเก็บคืนหลัง `idle_timeout` จนเหลือ `min_connections` (`min_connections` เป็น 0 ได้)


## ตัวอย่างการใช้ฐานข้อมูล
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        $default_uid = 29;
        $uid = $request->get('uid', $default_uid);
        $name = Db::table('users')->where('uid', $uid)->value('username');
        return response("hello $name");
    }
}
```

การใช้งานเหมือน Laravel ใช้ method `Db::table()` ในการทำงานกับฐานข้อมูล
