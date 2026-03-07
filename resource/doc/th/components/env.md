# คอมโพเนนต์ ENV vlucas/phpdotenv

## คำอธิบาย
`vlucas/phpdotenv` เป็นคอมโพเนนต์สำหรับโหลดตัวแปรสภาพแวดล้อม เพื่อแยกการตั้งค่าตามสภาพแวดล้อมต่าง ๆ เช่น สภาพแวดล้อมการพัฒนา สภาพแวดล้อมการทดสอบ เป็นต้น

## ที่อยู่โปรเจกต์

https://github.com/vlucas/phpdotenv
  
## การติดตั้ง
 
```php
composer require vlucas/phpdotenv
 ```
  
## การใช้งาน

### สร้างไฟล์ `.env` ในโฟลเดอร์หลักของโปรเจกต์
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### แก้ไขไฟล์การตั้งค่า
**config/database.php**
```php
return [
    // ฐานข้อมูลเริ่มต้น
    'default' => 'mysql',

    // การตั้งค่าหลักฐานข้อมูลต่างๆ
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **คำแนะนำ**
> แนะนำให้เพิ่มไฟล์ `.env` ในรายการ `.gitignore` เพื่อไม่ให้ถูกบันทึกลงคลังโค้ด ให้เพิ่มไฟล์ตัวอย่างการตั้งค่า `.env.example` ในคลังแทน เมื่อติดตั้งโปรเจกต์ ให้คัดลอก `.env.example` เป็น `.env` แล้วแก้ไขการตั้งค่าตามสภาพแวดล้อมปัจจุบัน โปรเจกต์จะโหลดการตั้งค่าต่างกันตามสภาพแวดล้อม

> **โปรดทราบ**
> `vlucas/phpdotenv` อาจมีบั๊กใน PHP เวอร์ชัน TS (Thread Safe) แนะนำให้ใช้เวอร์ชัน NTS (Non-Thread-Safe) สามารถตรวจสอบเวอร์ชัน PHP ปัจจุบันได้ด้วยคำสั่ง `php -v`

## ข้อมูลเพิ่มเติม

เข้าชม https://github.com/vlucas/phpdotenv
  
