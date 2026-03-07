# การจัดการเหตุการณ์
`webman/event` ให้กลไกเหตุการณ์ที่เหมาะเจาะ ช่วยให้ดำเนิน logic ทางธุรกิจได้โดยไม่ต้องแก้ไขโค้ด และแยกโมดูลออกจากกันได้ ตัวอย่างทั่วไป: เมื่อผู้ใช้ใหม่ลงทะเบียนสำเร็จ เพียงเผยแพร่เหตุการณ์กำหนดเองเช่น `user.register` แล้วแต่ละโมดูลจะรับเหตุการณ์นั้นและดำเนิน logic ที่เกี่ยวข้องได้

## การติดตั้ง
`composer require webman/event`

## การสมัครรับเหตุการณ์
การสมัครรับเหตุการณ์กำหนดค่าผ่านไฟล์ `config/event.php` แบบรวมศูนย์

```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...ฟังก์ชันประมวลผลเหตุการณ์อื่น ๆ...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...ฟังก์ชันประมวลผลเหตุการณ์อื่น ๆ...
    ]
];
```

**หมายเหตุ:**
- `user.register`, `user.logout` เป็นต้น คือชื่อเหตุการณ์ (สตริง) แนะนำให้ใช้คำตัวพิมพ์เล็กคั่นด้วยจุด (`.`)
- เหตุการณ์หนึ่งมีฟังก์ชันประมวลผลหลายตัวได้ โดยเรียกตามลำดับที่กำหนดค่า

## ฟังก์ชันประมวลผลเหตุการณ์
ฟังก์ชันประมวลผลจะเป็นเมทอดคลาส ฟังก์ชัน หรือ closure ก็ได้

ตัวอย่าง: สร้างคลาส `app/event/User.php` (สร้างโฟลเดอร์ถ้าไม่มี)

```php
<?php
namespace app\event;
class User
{
    function register($user)
    {
        var_export($user);
    }
 
    function logout($user)
    {
        var_export($user);
    }
}
```

## เผยแพร่เหตุการณ์
ใช้ `Event::dispatch($event_name, $data);` หรือ `Event::emit($event_name, $data);` เพื่อเผยแพร่เหตุการณ์ เช่น

```php
<?php
namespace app\controller;
use support\Request;
use Webman\Event\Event;
class User
{
    public function register(Request $request)
    {
        $user = [
            'name' => 'webman',
            'age' => 2
        ];
        Event::dispatch('user.register', $user);
    }
}
```

มีสองฟังก์ชันในการเผยแพร่: `Event::dispatch($event_name, $data);` และ `Event::emit($event_name, $data);` พารามิเตอร์เหมือนกัน ความต่าง: `emit` จับ exception ภายในอัตโนมัติ หากฟังก์ชันใดฟังก์ชันหนึ่งโยน exception ฟังก์ชันอื่นยังทำงานต่อ ส่วน `dispatch` ไม่จับ exception หากฟังก์ชันใดโยน exception การทำงานจะหยุดและ exception จะถูกส่งต่อไปข้างบน

> **เคล็ดลับ**
> พารามิเตอร์ `$data` เป็นข้อมูลใดก็ได้ เช่น แถว อินสแตนซ์คลาส สตริง เป็นต้น

## การฟังเหตุการณ์ด้วยอักขระตัวแทน
การลงทะเบียนด้วยอักขระตัวแทนให้ฟังหลายเหตุการณ์ด้วย listener เดียวกันได้ เช่น ใน `config/event.php`

```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```

ได้ชื่อเหตุการณ์ที่แน่นอนผ่านพารามิเตอร์ที่สอง `$event_data` ของฟังก์ชันประมวลผล:

```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // ชื่อเหตุการณ์ที่แน่นอน เช่น user.register, user.logout เป็นต้น
        var_export($user);
    }
}
```

## หยุดการกระจายเหตุการณ์
เมื่อฟังก์ชันประมวลผลคืนค่า `false` การกระจายเหตุการณ์นั้นจะหยุดลง

## การประมวลผลเหตุการณ์ด้วย closure
ฟังก์ชันประมวลผลอาจเป็นเมทอดคลาสหรือ closure ได้ เช่น

```php
<?php
return [
    'user.login' => [
        function($user){
            var_dump($user);
        }
    ]
];
```

## ดูเหตุการณ์และ listener
ใช้คำสั่ง `php webman event:list` เพื่อดูเหตุการณ์และ listener ที่กำหนดค่าในโปรเจกต์

## ขอบเขตการรองรับ
นอกจากโปรเจกต์หลักแล้ว [ปลั๊กอินพื้นฐาน](../plugin/base.md) และ [ปลั๊กอินแอป](../app/app.md) รองรับการกำหนดค่า `event.php` ด้วย
**ไฟล์ config ปลั๊กอินพื้นฐาน:** `config/plugin/vendor/ชื่อปลั๊กอิน/event.php`
**ไฟล์ config ปลั๊กอินแอป:** `plugin/ชื่อปลั๊กอิน/config/event.php`

## ข้อควรระวัง
การจัดการเหตุการณ์ไม่เป็นแบบอะซิงโครนัส และไม่เหมาะกับธุรกิจที่ช้า ธุรกิจที่ช้าควรใช้คิวข้อความ เช่น [webman/redis-queue](https://www.workerman.net/plugin/12)
