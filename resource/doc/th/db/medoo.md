# ฐานข้อมูล Medoo

[webman/medoo](https://github.com/webman-php/medoo) ขยาย [Medoo](https://medoo.in/) ด้วยการสนับสนุน connection pool และทำงานได้ทั้งในสภาพแวดล้อมคอร์รูทีนและไม่ใช่คอร์รูทีน การใช้งานเหมือนกับ Medoo

## การติดตั้ง
`composer require webman/medoo`

## การกำหนดค่าฐานข้อมูล Medoo
ตำแหน่งไฟล์การกำหนดค่า: `config/plugin/webman/medoo/database.php`

## การใช้ฐานข้อมูล Medoo
```php
<?php
namespace app\controller;

use support\Request;
use support\Medoo;

class Index
{
    public function index(Request $request)
    {
        $user = Medoo::get('user', '*', ['uid' => 1]);
        return json($user);
    }
}
```

> **คำแนะนำ**
> `Medoo::get('user', '*', ['uid' => 1]);`
> เทียบเท่ากับ
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## การกำหนดค่าฐานข้อมูล Medoo หลายฐานข้อมูล

**การกำหนดค่า**
เพิ่มการกำหนดค่าใหม่ใน `config/plugin/webman/medoo/database.php` โดยใช้ key อะไรก็ได้ ที่นี่ใช้ `other`

```php
<?php
return [
    'default' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [ // การกำหนดค่า connection pool
            'max_connections' => 5, // จำนวนการเชื่อมต่อสูงสุด
            'min_connections' => 1, // จำนวนการเชื่อมต่อขั้นต่ำ
            'wait_timeout' => 60,   // เวลารอสูงสุดเมื่อขอการเชื่อมต่อจาก pool; จะเกิด exception ถ้าเกิน
            'idle_timeout' => 3,    // เวลาว่างสูงสุดของการเชื่อมต่อใน pool; ที่เกินจะถูกปิดจนถึง min_connections
            'heartbeat_interval' => 50, // ช่วง heartbeat ของ pool (วินาที); แนะนำต่ำกว่า 60 วินาที
        ]
    ],
    // เพิ่มการกำหนดค่า 'other' ที่นี่
    'other' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [
            'max_connections' => 5,
            'min_connections' => 1,
            'wait_timeout' => 60,
            'idle_timeout' => 3,
            'heartbeat_interval' => 50,
        ],
    ],
];
```

## การใช้ฐานข้อมูล Medoo
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

ดู [เอกสารอย่างเป็นทางการของ Medoo](https://medoo.in/api/select)
