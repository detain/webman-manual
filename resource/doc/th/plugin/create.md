# กระบวนการสร้างและเผยแพร่ปลั๊กอินพื้นฐาน

## หลักการ
1. ยกตัวอย่างปลั๊กอิน cross-origin ปลั๊กอินประกอบด้วยสามส่วน: ไฟล์ middleware cross-origin ไฟล์คอนฟิก `middleware.php` และ `Install.php` ที่สร้างโดยคำสั่ง
2. ใช้คำสั่งในการแพ็คไฟล์ทั้งสามและเผยแพร่ผ่าน Composer
3. เมื่อผู้ใช้ติดตั้งปลั๊กอิน cross-origin ผ่าน Composer `Install.php` จะคัดลอกไฟล์ middleware และไฟล์คอนฟิกไปยัง `{โปรเจกต์หลัก}/config/plugin` เพื่อให้ webman โหลด ทำให้ cross-origin ทำงานโดยอัตโนมัติ
4. เมื่อผู้ใช้ลบปลั๊กอินผ่าน Composer `Install.php` จะลบไฟล์ middleware และไฟล์คอนฟิกที่เกี่ยวข้อง ทำให้ปลั๊กอินถูกลบโดยอัตโนมัติ

## ข้อกำหนด
1. ชื่อปลั๊กอินมีสองส่วน: `vendor` และ `ชื่อปลั๊กอิน` เช่น `webman/push` สอดคล้องกับชื่อแพ็คเกจ Composer
2. ไฟล์คอนฟิกปลั๊กอินเก็บใน `config/plugin/vendor/ชื่อปลั๊กอิน/` (คำสั่ง console จะสร้างโฟลเดอร์คอนฟิกโดยอัตโนมัติ) ถ้าปลั๊กอินไม่ต้องใช้คอนฟิก ต้องลบโฟลเดอร์ที่สร้างอัตโนมัติ
3. โฟลเดอร์คอนฟิกปลั๊กอินรองรับเฉพาะ: `app.php` (คอนฟิกหลัก) `bootstrap.php` (บูตโปรเซส) `route.php` (เส้นทาง) `middleware.php` (middleware) `process.php` (โปรเซสกำหนดเอง) `database.php` (ฐานข้อมูล) `redis.php` (Redis) `thinkorm.php` (thinkorm) webman จะรับรู้โดยอัตโนมัติ
4. การเข้าถึงคอนฟิก: `config('plugin.vendor.ชื่อปลั๊กอิน.ไฟล์คอนฟิก.รายการ');` เช่น `config('plugin.webman.push.app.app_key')`
5. ถ้าปลั๊กอินมีคอนฟิกฐานข้อมูลเอง: `illuminate/database` ใช้ `Db::connection('plugin.vendor.ชื่อปลั๊กอิน.การเชื่อมต่อ')` `thinkorm` ใช้ `Db::connect('plugin.vendor.ชื่อปลั๊กอิน.การเชื่อมต่อ')`
6. ถ้าปลั๊กอินจะใส่ไฟล์ธุรกิจใน `app/` ต้องไม่ชนกับโปรเจกต์หลักหรือปลั๊กอินอื่น
7. ปลั๊กอินควรหลีกเลี่ยงการคัดลอกไฟล์หรือโฟลเดอร์ไปยังโปรเจกต์หลัก เช่นปลั๊กอิน cross-origin คัดลอกเฉพาะคอนฟิก ไฟล์ middleware ให้อยู่ที่ `vendor/webman/cros/src`
8. แนะนำให้ใช้ PascalCase สำหรับ namespace ของปลั๊กอิน เช่น `Webman/Console`

## ตัวอย่าง

**ติดตั้ง `webman/console` command line**

`composer require webman/console`

### สร้างปลั๊กอิน

สมมติว่าชื่อปลั๊กอินที่จะสร้างคือ `foo/admin` (เป็นชื่อโปรเจกต์ที่จะเผยแพร่ผ่าน Composer ด้วย ต้องเป็นตัวพิมพ์เล็ก) รัน:

`php webman plugin:create --name=foo/admin`

จะได้ `vendor/foo/admin` (ไฟล์ปลั๊กอิน) และ `config/plugin/foo/admin` (คอนฟิก)

> หมายเหตุ
> `config/plugin/foo/admin` รองรับ: `app.php` `bootstrap.php` `route.php` `middleware.php` `process.php` `database.php` `redis.php` `thinkorm.php` รูปแบบเดียวกับ webman รวมเข้าระบบอัตโนมัติ
> ใช้ `plugin` เป็น prefix ในการเข้าถึง เช่น `config('plugin.foo.admin.app')`


### ส่งออกปลั๊กอิน

เมื่อพัฒนาเสร็จ รัน:

`php webman plugin:export --name=foo/admin`

ส่งออก

> คำอธิบาย
> การส่งออกจะคัดลอก `config/plugin/foo/admin` ไปยัง `vendor/foo/admin/src` และสร้าง `Install.php` สำหรับรันเมื่อติดตั้ง/ถอดปลั๊กอิน
> การติดตั้งเริ่มต้น: คัดลอกคอนฟิกจาก `vendor/foo/admin/src` ไปยัง `config/plugin` ของโปรเจกต์
> การถอดเริ่มต้น: ลบไฟล์คอนฟิกปลั๊กอินจาก `config/plugin` ของโปรเจกต์
> แก้ไข `Install.php` ได้เพื่อเพิ่มโลจิกกำหนดเองเมื่อติดตั้ง/ถอดปลั๊กอิน

### ส่งปลั๊กอิน
* สมมติว่ามีบัญชี [GitHub](https://github.com) และ [Packagist](https://packagist.org) แล้ว
* สร้าง repo `admin` บน [GitHub](https://github.com) และ push โค้ด เช่น `https://github.com/ชื่อผู้ใช้/admin`
* ไปที่ `https://github.com/ชื่อผู้ใช้/admin/releases/new` สร้าง release เช่น `v1.0.0`
* ที่ [Packagist](https://packagist.org) คลิก `Submit` ส่ง `https://github.com/ชื่อผู้ใช้/admin` เพื่อเผยแพร่ปลั๊กอิน

> **เคล็ดลับ**
> ถ้า Packagist แจ้งชื่อชน ให้เปลี่ยน vendor เช่น เปลี่ยน `foo/admin` เป็น `myfoo/admin`

เมื่ออัปเดต: push โค้ดขึ้น GitHub สร้าง release ใหม่ที่ `https://github.com/ชื่อผู้ใช้/admin/releases/new` จากนั้นคลิก `Update` ที่ `https://packagist.org/packages/foo/admin`

## เพิ่มคำสั่งในปลั๊กอิน
ปลั๊กอินบางตัวต้องมีคำสั่งกำหนดเอง เช่น ติดตั้ง `webman/redis-queue` แล้วโปรเจกต์จะได้คำสั่ง `redis-queue:consumer` รัน `php webman redis-queue:consumer send-mail` จะสร้างคลาส consumer `SendMail.php` ได้รวดเร็ว ช่วยในการพัฒนา

เพื่อเพิ่มคำสั่ง `foo-admin:add` ในปลั๊กอิน `foo/admin`:

### สร้างคำสั่ง

**สร้างไฟล์ `vendor/foo/admin/src/FooAdminAddCommand.php`**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = 'คำอธิบายคำสั่ง';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **หมายเหตุ**
> เพื่อหลีกเลี่ยงคำสั่งชนกันระหว่างปลั๊กอิน แนะนำรูปแบบ `vendor-plugin:คำสั่ง` เช่น คำสั่งทั้งหมดของปลั๊กอิน `foo/admin` ควรขึ้นต้นด้วย `foo-admin:` เช่น `foo-admin:add`

### เพิ่มคอนฟิก
**สร้าง `config/plugin/foo/admin/command.php`**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // เพิ่มตามต้องการ...
];
```

> **เคล็ดลับ**
> `command.php` ใช้ลงทะเบียนคำสั่งกำหนดเองของปลั๊กอิน แต่ละรายการคือคลาสคำสั่ง `webman/console` จะโหลดโดยอัตโนมัติ ดูเพิ่มเติมที่ [คำสั่ง console](console.md)

### รันการส่งออก
รัน `php webman plugin:export --name=foo/admin` เพื่อส่งออกปลั๊กอินและเผยแพร่บน Packagist เมื่อติดตั้ง `foo/admin` แล้วจะได้คำสั่ง `foo-admin:add` รัน `php webman foo-admin:add jerry` จะพิมพ์ `Admin add jerry`
