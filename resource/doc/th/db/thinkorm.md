# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) เป็นคอมโพเนนต์ฐานข้อมูลที่พัฒนาบน [top-think/think-orm](https://github.com/top-think/think-orm) รองรับ connection pool และทำงานได้ทั้งในสภาพแวดล้อม coroutine และ non-coroutine

## การติดตั้ง

`composer require -W webman/think-orm`

หลังติดตั้งต้อง restart (รีสตาร์ท) เท่านั้น (reload ไม่มีผล)

## ไฟล์กำหนดค่า

แก้ไขไฟล์กำหนดค่า `config/think-orm.php` ตามความต้องการจริงของคุณ

## เอกสาร

https://www.kancloud.cn/manual/think-orm

## การใช้งาน

```php
<?php
namespace app\controller;

use support\Request;
use support\think\Db;

class FooController
{
    public function get(Request $request)
    {
        $user = Db::table('user')->where('uid', '>', 1)->find();
        return json($user);
    }
}
```

## การสร้างโมเดล

โมเดล think-orm สืบทอดจาก `support\think\Model` ดังตัวอย่างด้านล่าง:

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * ตารางที่เกี่ยวข้องกับโมเดล
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * คีย์หลักของตาราง
     *
     * @var string
     */
    protected $pk = 'id';

}
```

คุณสามารถใช้คำสั่งด้านล่างสร้างโมเดล think-orm ได้เช่นกัน:

```
php webman make:model ชื่อตาราง
```

> **คำแนะนำ**
> คำสั่งนี้ต้องมี `webman/console` ติดตั้งก่อน: `composer require webman/console ^1.2.13`

> **ข้อควรระวัง**
> ถ้าคำสั่ง make:model ตรวจพบว่าโปรเจ็กต์หลักใช้ `illuminate/database` จะสร้างไฟล์โมเดลจาก Illuminate แทน think-orm ในกรณีนี้ให้เพิ่มพารามิเตอร์ `tp`: `php webman make:model ชื่อตาราง tp` (ถ้าไม่ทำงานให้อัปเกรด `webman/console`)
