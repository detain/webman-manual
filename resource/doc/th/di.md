# การฉีดพึ่งพาอัตโนมัติ

ใน webman การฉีดพึ่งพาอัตโนมัติเป็นฟีเจอร์ตัวเลือก และปิดโดยค่าเริ่มต้น ถ้าต้องการการฉีดพึ่งพาอัตโนมัติ แนะนำให้ใช้ [php-di](https://php-di.org/doc/getting-started.html) ด้านล่างอธิบายการใช้ `php-di` กับ webman

## การติดตั้ง

```
composer require php-di/php-di:^7.0
```

แก้ไขการตั้งค่า `config/container.php` เนื้อหาสุดท้ายต้องเป็นดังนี้:

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> ไฟล์ `config/container.php` ต้องส่งคืน instance คอนเทนเนอร์ที่ปฏิบัติตามข้อกำหนด PSR-11 ในตอนท้าย หากไม่ต้องการใช้ `php-di` สามารถสร้างและส่งคืน instance คอนเทนเนอร์ที่ปฏิบัติตาม PSR-11 อื่นที่นี่ได้ การตั้งค่าเริ่มต้นให้เฉพาะฟังก์ชันพื้นฐานของ webman container

## การฉีดผ่าน constructor

สร้างไฟล์ `app/service/Mailer.php` (สร้างโฟลเดอร์หากไม่มี) ด้วยเนื้อหาดังนี้:

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // โค้ดส่งอีเมลละไว้
    }
}
```

เนื้อหาของ `app/controller/UserController.php`:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{

    public function __construct(private Mailer $mailer)
    {
    }

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'สวัสดีและยินดีต้อนรับ!');
        return response('ok');
    }
}
```

โดยปกติ ต้องใช้โค้ดต่อไปนี้เพื่อสร้าง instance `app\controller\UserController`:

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

เมื่อใช้ `php-di` นักพัฒนไม่ต้องสร้าง instance `Mailer` ใน controller เอง — webman จะทำให้อัตโนมัติ ถ้าขณะสร้าง instance `Mailer` มีการพึ่งพาคลาสอื่น webman จะสร้างและฉีดให้อัตโนมัติเช่นกัน ไม่ต้องดำเนินการเริ่มต้นใด ๆ จากนักพัฒนา

> **หมายเหตุ**
> เฉพาะ instance ที่สร้างโดย framework หรือ `php-di` เท่านั้นที่รองรับการฉีดพึ่งพาอัตโนมัติ Instance ที่สร้างด้วย `new` เองไม่สามารถใช้ได้ หากต้องการฉีด ให้ใช้ interface `support\Container` แทน `new` เช่น:

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// Instance ที่สร้างด้วย new ไม่รองรับการฉีดพึ่งพา
$user_service = new UserService;
// Instance ที่สร้างด้วย new ไม่รองรับการฉีดพึ่งพา
$log_service = new LogService($path, $name);

// Instance ที่สร้างด้วย Container รองรับการฉีดพึ่งพา
$user_service = Container::get(UserService::class);
// Instance ที่สร้างด้วย Container รองรับการฉีดพึ่งพา
$log_service = Container::make(LogService::class, [$path, $name]);
```

## การฉีดผ่าน Attributes

นอกจากฉีดผ่าน constructor แล้ว ยังใช้การฉีดผ่าน attributes ได้ ต่อจากตัวอย่างก่อนหน้า แก้ไข `app\controller\UserController` ดังนี้:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private Mailer $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'สวัสดีและยินดีต้อนรับ!');
        return response('ok');
    }
}
```

ตัวอย่างนี้ใช้ attribute `#[Inject]` เพื่อฉีด และฉีด instance เข้า member variable โดยอัตโนมัติตามประเภทออบเจ็กต์ ผลลัพธ์เหมือนการฉีดผ่าน constructor แต่โค้ดกระชับขึ้น

> **หมายเหตุ**
> webman ไม่รองรับการฉีดพารามิเตอร์ controller ก่อนเวอร์ชัน 1.4.6 เช่น โค้ดต่อไปนี้ไม่รองรับเมื่อ webman<=1.4.6:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // การฉีดพารามิเตอร์ controller ไม่รองรับก่อนเวอร์ชัน 1.4.6
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'สวัสดีและยินดีต้อนรับ!');
        return response('ok');
    }
}
```

## การฉีด constructor แบบกำหนดเอง

บางครั้งพารามิเตอร์ constructor อาจไม่ใช่ instance ของคลาสแต่เป็นสตริง ตัวเลข อาเรย์ หรือข้อมูลอื่น เช่น constructor Mailer อาจต้องการ IP และพอร์ตเซิร์ฟเวอร์ SMTP:

```php
<?php
namespace app\service;

class Mailer
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // โค้ดส่งอีเมลละไว้
    }
}
```

กรณีนี้ไม่สามารถใช้การฉีดอัตโนมัติผ่าน constructor โดยตรง เพราะ `php-di` ไม่ทราบค่า `$smtp_host` และ `$smtp_port` ในกรณีนี้ลองใช้การฉีดแบบกำหนดเอง

เพิ่มโค้ดต่อไปนี้ใน `config/dependence.php` (สร้างไฟล์หากไม่มี):

```php
return [
    // ... การตั้งค่าอื่นละไว้

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

เมื่อการฉีดพึ่งพาต้องการ instance ของ `app\service\Mailer` จะใช้ instance ที่สร้างในการตั้งค่านี้โดยอัตโนมัติ

สังเกตว่า `config/dependence.php` ใช้ `new` เพื่อสร้าง instance คลาส `Mailer` ในตัวอย่างนี้ไม่มีปัญหา แต่ถ้าคลาส `Mailer` พึ่งพาคลาสอื่นหรือใช้การฉีดผ่าน attributes ข้างใน การเริ่มต้นด้วย `new` จะไม่ทำการฉีดพึ่งพาอัตโนมัติ วิธีแก้คือใช้การฉีด interface แบบกำหนดเอง และเริ่มต้นคลาสผ่าน `Container::get(ชื่อคลาส)` หรือ `Container::make(ชื่อคลาส, [พารามิเตอร์ constructor])`

## การฉีด interface แบบกำหนดเอง

ในโปรเจกต์จริง ควรเขียนโปรแกรมต่อ interface มากกว่าคลาสจริง เช่น `app\controller\UserController` ควรพึ่งพา `app\service\MailerInterface` แทน `app\service\Mailer`

กำหนด interface `MailerInterface`:

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

กำหนดการ implement ของ `MailerInterface`:

```php
<?php
namespace app\service;

class Mailer implements MailerInterface
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // โค้ดส่งอีเมลละไว้
    }
}
```

ใช้ interface `MailerInterface` แทนการ implement จริง:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\MailerInterface;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private MailerInterface $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'สวัสดีและยินดีต้อนรับ!');
        return response('ok');
    }
}
```

กำหนดการ implement ของ `MailerInterface` ใน `config/dependence.php`:

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

เมื่อธุรกิจต้องใช้ interface `MailerInterface` จะใช้การ implement `Mailer` โดยอัตโนมัติ

> ข้อดีของการเขียนโปรแกรมต่อ interface คือ เมื่อต้องเปลี่ยน component ไม่ต้องแก้โค้ดธุรกิจ แค่เปลี่ยนการ implement จริงใน `config/dependence.php` ยังมีประโยชน์มากสำหรับการทดสอบหน่วย

## การฉีดแบบกำหนดเองอื่น ๆ

นอกจากกำหนดการพึ่งพาคลาส `config/dependence.php` ยังกำหนดค่าอื่น เช่น สตริง ตัวเลข อาเรย์ ได้

เช่น ถ้า `config/dependence.php` กำหนดดังนี้:

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

สามารถใช้ `#[Inject]` เพื่อฉีด `smtp_host` และ `smtp_port` เข้า properties ของคลาส:

```php
<?php
namespace app\service;

use DI\Attribute\Inject;

class Mailer
{
    #[Inject("smtp_host")]
    private $smtpHost;

    #[Inject("smtp_port")]
    private $smtpPort;

    public function mail($email, $content)
    {
        // โค้ดส่งอีเมลละไว้
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // จะแสดง 192.168.1.11:25
    }
}
```

# การโหลดแบบเฉื่อย (Lazy Loading)

> การโหลดแบบเฉื่อยเป็นรูปแบบการออกแบบที่เลื่อนการสร้างหรือเริ่มต้นออบเจ็กต์จนกว่าจะใช้จริง

ฟีเจอร์นี้ต้องใช้ dependency เพิ่มเติม แพ็กเกจต่อไปนี้เป็น fork ของ `ocramius/proxy-manager` รีโพซิทอรีดั้งเดิมไม่รองรับ PHP 8

```
composer require friendsofphp/proxy-manager-lts
```

วิธีใช้:

```php
<?php

use DI\Attribute\Injectable;
use DI\Attribute\Inject;

#[Injectable(lazy: true)]
class MyClass
{
    private string $name;

    public function __construct()
    {
        echo "MyClass ถูกสร้าง instance\n";
        $this->name = "Lazy Loaded Object";
    }

    public function getName(): string
    {
        return $this->name;
    }
}

class Controller
{
    #[Inject]
    public MyClass $myClass;

    public function getClass()
    {
        echo "ชื่อคลาสพร็อกซี่: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

ผลลัพธ์:

```
ชื่อคลาสพร็อกซี่: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass ถูกสร้าง instance
name: Lazy Loaded Object
```

ตัวอย่างนี้แสดงว่าเมื่อคลาสที่ประกาศด้วย attribute `#[Injectable]` ถูกฉีด จะสร้างคลาสพร็อกซี่ก่อน คลาสจริงจะถูกสร้าง instance ก็ต่อเมื่อเรียกเมธอดใด ๆ ของมัน

# การพึ่งพาแบบวน (Circular Dependencies)

การพึ่งพาแบบวนเกิดขึ้นเมื่อหลายคลาสพึ่งพาซึ่งกันและกัน เป็นวงจรการพึ่งพาปิด

- การพึ่งพาแบบวนโดยตรง
  - โมดูล A พึ่งพาโมดูล B โมดูล B พึ่งพาโมดูล A
  - เป็นวงจร: A → B → A

- การพึ่งพาแบบวนโดยอ้อม
  - มีหลายโมดูลในวงจรการพึ่งพา
  - เช่น A → B → C → A

เมื่อใช้การฉีดผ่าน attributes `php-di` จะตรวจจับการพึ่งพาแบบวนโดยอัตโนมัติและโยน exception หากจำเป็น ให้ใช้แนวทางต่อไปนี้แทน:

```php
class userController
{

    // ลบบรรทัดนี้
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## ข้อมูลเพิ่มเติม

โปรดดู [เอกสาร php-di](https://php-di.org/doc/getting-started.html)
