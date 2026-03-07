# การจัดการเซสชัน webman

## ตัวอย่าง
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

รับอินสแตนซ์ `Workerman\Protocols\Http\Session` ผ่าน `$request->session();` และใช้เมธอดของมันเพื่อเพิ่ม แก้ไข หรือลบข้อมูลเซสชัน

> **หมายเหตุ**
> ข้อมูลเซสชันจะถูกบันทึกอัตโนมัติเมื่อวัตถุเซสชันถูกทำลาย
> การเก็บวัตถุเซสชันในตัวแปรโกลบอลจะป้องกันไม่ให้วัตถุถูกทำลาย ทำให้ไม่มีการบันทึกอัตโนมัติ ในกรณีนี้ต้องเรียก `$session->save()` ด้วยตัวเองเพื่อบันทึกข้อมูล

## รับข้อมูลเซสชันทั้งหมด
```php
$session = $request->session();
$all = $session->all();
```
คืนค่าเป็นอาร์เรย์ หากไม่มีข้อมูลเซสชัน จะคืนค่าอาร์เรย์ว่าง


## รับค่าจากเซสชัน
```php
$session = $request->session();
$name = $session->get('name');
```
หากข้อมูลไม่มี จะคืนค่า null

คุณสามารถส่งค่าเริ่มต้นเป็นอาร์กิวเมนต์ที่สองของเมธอด `get` ได้ หากไม่พบค่าในอาร์เรย์เซสชัน จะคืนค่าเริ่มต้น ตัวอย่าง:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## บันทึกข้อมูลเซสชัน
ใช้เมธอด `set` เพื่อบันทึกข้อมูลหนึ่งรายการ
```php
$session = $request->session();
$session->set('name', 'tom');
```
เมธอด `set` ไม่คืนค่า ข้อมูลเซสชันจะถูกบันทึกอัตโนมัติเมื่อวัตถุเซสชันถูกทำลาย

ใช้เมธอด `put` เพื่อบันทึกหลายค่า
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
ในทำนองเดียวกัน เมธอด `put` ไม่คืนค่า

## ลบข้อมูลเซสชัน
ใช้เมธอด `forget` เพื่อลบข้อมูลเซสชันหนึ่งรายการหรือมากกว่า
```php
$session = $request->session();
// ลบหนึ่งรายการ
$session->forget('name');
// ลบหลายรายการ
$session->forget(['name', 'age']);
```

ระบบยังมีเมธอด `delete` ด้วย ซึ่งต่างจาก `forget` ตรงที่ลบได้เพียงหนึ่งรายการ
```php
$session = $request->session();
// เท่ากับ $session->forget('name');
$session->delete('name');
```


## รับและลบค่าจากเซสชัน
```php
$session = $request->session();
$name = $session->pull('name');
```
ให้ผลเหมือนกับโค้ดต่อไปนี้:
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
หากไม่มีค่าเซสชันที่ตรงกัน จะคืนค่า null


## ลบข้อมูลเซสชันทั้งหมด
```php
$request->session()->flush();
```
ไม่คืนค่า เมื่อวัตถุเซสชันถูกทำลาย ข้อมูลเซสชันจะถูกลบออกจากพื้นที่จัดเก็บโดยอัตโนมัติ


## ตรวจสอบว่ามีค่าเซสชันหรือไม่
```php
$session = $request->session();
$has = $session->has('name');
```
คืนค่า false หากค่าเซสชันไม่มีหรือเป็น null มิฉะนั้นคืนค่า true

```php
$session = $request->session();
$has = $session->exists('name');
```
โค้ดด้านบนก็ใช้ตรวจสอบว่ามีค่าเซสชันหรือไม่เช่นกัน ความแตกต่างคือ `exists` จะคืนค่า true แม้เมื่อค่าเป็น null

## ฟังก์ชันช่วย session()

webman มีฟังก์ชันช่วย `session()` สำหรับทำงานแบบเดียวกัน
```php
// รับอินสแตนซ์เซสชัน
$session = session();
// เท่ากับ
$session = $request->session();

// รับค่า
$value = session('key', 'default');
// เท่ากับ
$value = session()->get('key', 'default');
// เท่ากับ
$value = $request->session()->get('key', 'default');

// กำหนดค่าให้เซสชัน
session(['key1'=>'value1', 'key2' => 'value2']);
// เท่ากับ
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// เท่ากับ
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## ไฟล์การตั้งค่า
ไฟล์การตั้งค่าเซสชันอยู่ที่ `config/session.php` มีรูปแบบดังนี้:
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class หรือ RedisSessionHandler::class หรือ RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // เมื่อ handler เป็น FileSessionHandler::class ค่าเป็น 'file'
    // เมื่อ handler เป็น RedisSessionHandler::class ค่าเป็น 'redis'
    // เมื่อ handler เป็น RedisClusterSessionHandler::class ค่าเป็น 'redis_cluster' (คลัสเตอร์ Redis)
    'type'    => 'file',

    // handler ที่ต่างกันใช้การตั้งค่าที่ต่างกัน
    'config' => [
        // การตั้งค่าเมื่อ type เป็น 'file'
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // การตั้งค่าเมื่อ type เป็น 'redis'
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // ชื่อคุกกี้เก็บ session_id
    'auto_update_timestamp' => false,  // รีเฟรชเซสชันอัตโนมัติหรือไม่ ค่าเริ่มต้น: ปิด
    'lifetime' => 7*24*60*60,          // อายุเซสชัน
    'cookie_lifetime' => 365*24*60*60, // อายุคุกกี้เก็บ session_id
    'cookie_path' => '/',              // เส้นทางคุกกี้ session_id
    'domain' => '',                    // โดเมนคุกกี้ session_id
    'http_only' => true,               // เปิดใช้ httpOnly หรือไม่ ค่าเริ่มต้น: เปิด
    'secure' => false,                 // เปิดใช้เซสชันเฉพาะ HTTPS ค่าเริ่มต้น: ปิด
    'same_site' => '',                 // ป้องกัน CSRF และการติดตามผู้ใช้ ค่า: strict/lax/none
    'gc_probability' => [1, 1000],     // ความน่าจะเป็นในการล้างเซสชัน
];
```

## ความปลอดภัย
ไม่แนะนำให้เก็บอินสแตนซ์คลาสลงในเซสชันโดยตรง โดยเฉพาะจากแหล่งที่ไม่น่าเชื่อถือ การดีซีเรียลไลซ์อาจก่อให้เกิดความเสี่ยงด้านความปลอดภัย

