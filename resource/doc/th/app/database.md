# ฐานข้อมูล

เนื่องจากปลั๊กอินส่วนใหญ่ติดตั้ง [webman-admin](https://www.workerman.net/plugin/82) ดังนั้นจึงแนะนำให้ใช้การตั้งค่าฐานข้อมูลของ `webman-admin` โดยตรง

โมเดลที่คลาสพื้นฐานคือ `plugin\admin\app\model\Base` จะใช้การตั้งค่าฐานข้อมูลของ webman-admin โดยอัตโนมัติ
```php
<?php

namespace plugin\foo\app\model;

use plugin\admin\app\model\Base;

class Orders extends Base
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'foo_orders';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $primaryKey = 'id';
    
}
```

คุณยังเข้าถึงฐานข้อมูล webman-admin ผ่าน `plugin.admin.mysql` ได้ เช่น:

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## ใช้ฐานข้อมูลของตัวเอง

ปลั๊กอินสามารถเลือกใช้ฐานข้อมูลของตัวเองได้ เช่น เนื้อหาของ `plugin/foo/config/database.php`:

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql คือชื่อการเชื่อมต่อ
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'ฐานข้อมูล',
            'username'    => 'ชื่อผู้ใช้',
            'password'    => 'รหัสผ่าน',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin คือชื่อการเชื่อมต่อ
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'ฐานข้อมูล',
            'username'    => 'ชื่อผู้ใช้',
            'password'    => 'รหัสผ่าน',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

รูปแบบการอ้างอิงคือ `Db::connection('plugin.{ปลั๊กอิน}.{ชื่อการเชื่อมต่อ}');` เช่น:

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

หากต้องการใช้ฐานข้อมูลของโปรเจกต์หลัก สามารถเรียกใช้โดยตรงได้ เช่น:

```php
use support\Db;
Db::table('user')->first();
// สมมติว่าโปรเจกต์หลักมีการตั้งค่าการเชื่อมต่อ admin ด้วย
Db::connection('admin')->table('admin')->first();
```

#### กำหนดค่าฐานข้อมูลสำหรับ Model

คุณสามารถสร้างคลาส Base สำหรับ Model และตั้งคุณสมบัติ `$connection` เพื่อใช้การเชื่อมต่อฐานข้อมูลของปลั๊กอินเอง:

```php
<?php

namespace plugin\foo\app\model;

use DateTimeInterface;
use support\Model;

class Base extends Model
{
    /**
     * @var string
     */
    protected $connection = 'plugin.foo.mysql';

}
```

ดังนั้นโมเดลทั้งหมดในปลั๊กอินที่สืบทอดจาก Base จะใช้ฐานข้อมูลของปลั๊กอินเองโดยอัตโนมัติ

## นำเข้าฐานข้อมูลอัตโนมัติ
การรัน `php webman app-plugin:create foo` จะสร้างปลั๊กอิน foo โดยอัตโนมัติ รวมถึง `plugin/foo/api/Install.php` และ `plugin/foo/install.sql`

> **เคล็ดลับ**
> หากไฟล์ install.sql ไม่ถูกสร้าง ให้ลองอัปเกรด `webman/console`: `composer require webman/console ^1.3.6`

#### นำเข้าฐานข้อมูลเมื่อติดตั้งปลั๊กอิน
เมื่อติดตั้งปลั๊กอิน เมธอด `install` ใน Install.php จะถูกเรียกใช้ เมธอดนี้จะรันคำสั่ง SQL ใน `install.sql` โดยอัตโนมัติ เพื่อนำเข้าตารางฐานข้อมูล

เนื้อหาของ `install.sql` คือการสร้างตารางและคำสั่ง SQL ที่แก้ไขตารางในอดีต โปรดทราบว่าทุกคำสั่งต้องลงท้ายด้วย `;` เช่น:
```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'คีย์หลัก',
  `order_id` varchar(50) NOT NULL COMMENT 'ID คำสั่งซื้อ',
  `user_id` int NOT NULL COMMENT 'ID ผู้ใช้',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'ยอดชำระ',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='คำสั่งซื้อ';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'คีย์หลัก',
  `name` varchar(50) NOT NULL COMMENT 'ชื่อ',
  `price` int NOT NULL COMMENT 'ราคา',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='สินค้า';
```

**เปลี่ยนการเชื่อมต่อฐานข้อมูล**

ฐานข้อมูลเป้าหมายของการนำเข้า `install.sql` คือฐานข้อมูลของ webman-admin ตามค่าเริ่มต้น หากต้องการนำเข้าไปยังฐานข้อมูลอื่น ให้แก้ไขคุณสมบัติ `$connection` ใน `Install.php` เช่น:
```php
<?php

class Install
{
    // ระบุฐานข้อมูลของปลั๊กอินเอง
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**ทดสอบ**

รัน `php webman app-plugin:install foo` เพื่อติดตั้งปลั๊กอิน แล้วตรวจสอบฐานข้อมูล จะพบว่าตาราง `foo_orders` และ `foo_goods` ถูกสร้างแล้ว

#### เปลี่ยนโครงสร้างตารางเมื่ออัปเกรดปลั๊กอิน
บางครั้งการอัปเกรดปลั๊กอินต้องสร้างตารางใหม่หรือเปลี่ยนโครงสร้างตาราง สามารถเพิ่มคำสั่งที่ตรงกันลงท้าย `install.sql` ได้โดยตรง โปรดทราบว่าทุกคำสั่งลงท้ายด้วย `;` เช่น เพิ่มตาราง `foo_user` และเพิ่มคอลัมน์ `status` ให้ตาราง `foo_orders`
```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'คีย์หลัก',
  `order_id` varchar(50) NOT NULL COMMENT 'ID คำสั่งซื้อ',
  `user_id` int NOT NULL COMMENT 'ID ผู้ใช้',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'ยอดชำระ',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='คำสั่งซื้อ';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'คีย์หลัก',
 `name` varchar(50) NOT NULL COMMENT 'ชื่อ',
 `price` int NOT NULL COMMENT 'ราคา',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='สินค้า';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'คีย์หลัก',
 `name` varchar(50) NOT NULL COMMENT 'ชื่อ'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='ผู้ใช้';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT 'สถานะ';
```

เมื่ออัปเกรด เมธอด `update` ใน Install.php จะถูกเรียกใช้ เมธอดนี้จะรันคำสั่งใน `install.sql` เช่นกัน ถ้ามีคำสั่งใหม่จะรันคำสั่งใหม่ ถ้าเป็นคำสั่งเก่าจะข้าม เพื่อให้การแก้ไขฐานข้อมูลเมื่ออัปเกรดสำเร็จ

#### ลบฐานข้อมูลเมื่อถอดการติดตั้งปลั๊กอิน
เมื่อถอดการติดตั้งปลั๊กอิน เมธอด `uninstall` ใน Install.php จะถูกเรียก มันจะวิเคราะห์คำสั่ง CREATE TABLE ใน `install.sql` โดยอัตโนมัติ และลบตารางเหล่านั้น เพื่อลบตารางฐานข้อมูลเมื่อถอดการติดตั้งปลั๊กอิน
หากต้องการให้เมื่อถอดการติดตั้งรันเฉพาะ `uninstall.sql` ของตัวเอง ไม่รันการลบตารางอัตโนมัติ สร้าง `plugin/ชื่อปลั๊กอิน/uninstall.sql` เท่านั้น เมธอด `uninstall` จะรันเฉพาะคำสั่งในไฟล์ `uninstall.sql`
