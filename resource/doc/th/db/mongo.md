# MongoDB

webman ใช้ [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) เป็นคอมโพเนนต์ MongoDB โดยค่าเริ่มต้น ซึ่งถูกแยกออกมาจากโครงการ Laravel และใช้ได้อย่างเดียวกันกับ Laravel

ก่อนที่จะใช้ `mongodb/laravel-mongodb` คุณต้องติดตั้งส่วนขยาย mongodb สำหรับ `php-cli` ก่อน

> **โปรดทราบ**
> ใช้คำสั่ง `php -m | grep mongodb` เพื่อตรวจสอบว่า `php-cli` ติดตั้งส่วนขยาย mongodb หรือยัง โปรดทราบว่า แม้ว่าคุณจะติดตั้งส่วนขยาย mongodb สำหรับ `php-fpm` แล้ว ก็ไม่ได้หมายความว่าคุณสามารถใช้งานได้สำหรับ `php-cli` เพราะ `php-cli` และ `php-fpm` เป็นโปรแกรมที่แตกต่างกัน และอาจจะใช้การกำหนดค่า `php.ini` ที่แตกต่างกัน ใช้คำสั่ง `php --ini` เพื่อตรวจสอบว่าคุณกำลังใช้ไฟล์การกำหนดค่า `php.ini` ที่ไหนสำหรับ `php-cli`

## การติดตั้ง

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

หลังจากติดตั้งแล้วจำเป็นต้องทำการ restart (reload ไม่สามารถใช้งานได้)

## การกำหนดค่า

เพิ่ม `mongodb` connection ใน `config/database.php` เช่นเดียวกับตัวอย่างต่อไปนี้:
```php
return [

    'default' => 'mysql',

    'connections' => [

         ...การกำหนดอื่นๆ ถูกข้ามไป...

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => '127.0.0.1',
            'port'     =>  27017,
            'database' => 'test',
            'username' => null,
            'password' => null,
            'options' => [
                // ที่นี่คุณสามารถส่งการตั้งค่าเพิ่มเติมไปยัง Mongo Driver Manager
                // ดูที่ https://www.php.net/manual/en/mongodb-driver-manager.construct.php ภายใต้ "Uri Options" เพื่อดูรายการของพารามิเตอร์ที่คุณสามารถใช้ได้ทั้งหมด

                'appname' => 'homestead'
            ],
        ],
    ],
];
```

## ตัวอย่าง
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        Db::connection('mongodb')->table('test')->insert([1,2,3]);
        return json(Db::connection('mongodb')->table('test')->get());
    }
}
```

## ตัวอย่าง Model
```php
<?php
namespace app\model;

use DateTimeInterface;
use support\MongoModel as Model;

class Test extends Model
{
    protected $connection = 'mongodb';

    protected $table = 'test';

    public $timestamps = true;

    /**
     * @param DateTimeInterface $date
     * @return string
     */
    protected function serializeDate(DateTimeInterface $date): string
    {
        return $date->format('Y-m-d H:i:s');
    }
}
```

## สำหรับข้อมูลเพิ่มเติมโปรดเข้าไปที่

https://github.com/mongodb/laravel-mongodb
