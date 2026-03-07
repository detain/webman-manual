# หลายภาษา

การรองรับหลายภาษใช้คอมโพเนนต์ [symfony/translation](https://github.com/symfony/translation)

## การติดตั้ง
```
composer require symfony/translation
```

## สร้างแพ็กเกจภาษา
webman จะเก็บแพ็กเกจภาษาไว้ในโฟลเดอร์ `resource/translations` โดยค่าเริ่มต้น (สร้างโฟลเดอร์หากยังไม่มี) หากต้องการเปลี่ยนโฟลเดอร์ ให้ตั้งค่าใน `config/translation.php`
แต่ละภาษาจะมีโฟลเดอร์ย่อย แยกตามภาษา คำจำกัดความภาษาเก็บไว้ใน `messages.php` โดยค่าเริ่มต้น ตัวอย่าง:
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

ไฟล์ภาษาทั้งหมดจะคืนค่าเป็นอาร์เรย์ เช่น:
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## การกำหนดค่า

`config/translation.php`

```php
return [
    // ภาษาเริ่มต้น
    'locale' => 'zh_CN',
    // ภาษาสำรอง: หากไม่พบคำแปลในภาษาปัจจุบัน จะลองใช้คำแปลจากภาษาสำรอง
    'fallback_locale' => ['zh_CN', 'en'],
    // โฟลเดอร์เก็บไฟล์ภาษา
    'path' => base_path() . '/resource/translations',
];
```

## การแปล

ใช้เมธอด `trans()` สำหรับการแปล

สร้างไฟล์ภาษา `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 世界!',
];
```

สร้างไฟล์ `app/controller/UserController.php`:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        $hello = trans('hello'); // 你好 世界!
        return response($hello);
    }
}
```

เมื่อเข้าถึง `http://127.0.0.1:8787/user/get` จะได้ "你好 世界!"

## เปลี่ยนภาษาเริ่มต้น

ใช้เมธอด `locale()` เพื่อสลับภาษา

เพิ่มไฟล์ภาษา `resource/translations/en/messages.php`:
```php
return [
    'hello' => 'hello world!',
];
```

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // สลับภาษา
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
เมื่อเข้าถึง `http://127.0.0.1:8787/user/get` จะได้ "hello world!"

ยังสามารถใช้พารามิเตอร์ตัวที่ 4 ของฟังก์ชัน `trans()` เพื่อสลับภาษาชั่วคราวได้ เช่น ตัวอย่างด้านบนเทียบเท่ากับ:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // พารามิเตอร์ตัวที่ 4 สลับภาษา
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## ตั้งค่าภาษาอย่างชัดเจนสำหรับแต่ละคำขอ
translation เป็นซิงเกิลตัน หมายความว่าคำขอทั้งหมดแชร์อินสแตนซ์เดียวกัน หากคำขอใดใช้ `locale()` ตั้งค่าภาษาเริ่มต้น จะส่งผลต่อคำขอทั้งหมดถัดไปในโปรเซสนั้น ดังนั้นควรตั้งค่าภาษาอย่างชัดเจนสำหรับแต่ละคำขอ เช่น ใช้มิดเดิลแวร์ต่อไปนี้:

สร้างไฟล์ `app/middleware/Lang.php` (สร้างโฟลเดอร์หากยังไม่มี):
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Lang implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        locale(session('lang', 'zh_CN'));
        return $handler($request);
    }
}
```

เพิ่มมิดเดิลแวร์ส่วนกลางใน `config/middleware.php`:
```php
return [
    // มิดเดิลแวร์ส่วนกลาง
    '' => [
        // ... มิดเดิลแวร์อื่น ๆ ข้ามไป
        app\middleware\Lang::class,
    ]
];
```


## ใช้ตัวยึด
บางครั้งข้อความมีตัวแปรที่ต้องแปล เช่น
```php
trans('hello ' . $name);
```
ในกรณีนี้ใช้ตัวยึดจัดการ

แก้ไข `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 %name%!',
];
```
เมื่อแปล ส่งค่ารอบตัวยึดผ่านพารามิเตอร์ตัวที่ 2:
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## การจัดการพหูพจน์
บางภาษาประโยคเปลี่ยนตามจำนวน เช่น `There is %count% apple` ถูกต้องเมื่อ `%count%` เป็น 1 แต่ผิดเมื่อมากกว่า 1

ในกรณีนี้ใช้ **ท่อ** (`|`) เพื่อแสดงรูปแบบพหูพจน์

เพิ่ม `apple_count` ในไฟล์ภาษา `resource/translations/en/messages.php`:
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

ยังสามารถกำหนดช่วงตัวเลขเพื่อสร้างกฎพหูพจน์ที่ซับซ้อนขึ้นได้:
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## ระบุไฟล์ภาษา

ชื่อไฟล์เริ่มต้นคือ `messages.php` แต่สามารถสร้างไฟล์ภาษาอื่นได้

สร้างไฟล์ภาษา `resource/translations/zh_CN/admin.php`:
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

ระบุไฟล์ภาษาผ่านพารามิเตอร์ตัวที่ 3 ของ `trans()` (ไม่ต้องใส่ส่วนขยาย `.php`):
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## ข้อมูลเพิ่มเติม
ดู [คู่มือ symfony/translation](https://symfony.com/doc/current/translation.html)
