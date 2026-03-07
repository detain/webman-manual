# แคช

[webman/cache](https://github.com/webman-php/cache) เป็นคอมโพเนนต์แคชที่พัฒนาบนฐาน [symfony/cache](https://github.com/symfony/cache) รองรับทั้งสภาพแวดล้อม coroutine และ non-coroutine และรองรับ connection pool

## การติดตั้ง

```php
composer require -W webman/cache
```

## ตัวอย่าง
```php
<?php
namespace app\controller;

use support\Request;
use support\Cache;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Cache::set($key, rand());
        return response(Cache::get($key));
    }
}
```

## ตำแหน่งไฟล์กำหนดค่า
ไฟล์กำหนดค่าอยู่ที่ `config/cache.php` หากไม่มีให้สร้างด้วยตนเอง

## เนื้อหาไฟล์กำหนดค่า
```php
<?php
return [
    'default' => 'file',
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default'
        ],
        'array' => [
            'driver' => 'array'
        ],
        'apcu' => [
            'driver' => 'apcu'
        ]
    ]
];
```
`stores.driver` รองรับ 4 ไดรเวอร์: **file**, **redis**, **array** และ **apcu**

#### ไดรเวอร์ file
เป็นไดรเวอร์เริ่มต้น ไม่พึ่งพาคอมโพเนนต์ภายนอก รองรับการแชร์แคชระหว่างกระบวนการ ไม่รองรับการแชร์ระหว่างหลายเซิร์ฟเวอร์

#### ไดรเวอร์ array
จัดเก็บในหน่วยความจำ ประสิทธิภาพดีที่สุด แต่ใช้หน่วยความจำ ไม่รองรับการแชร์ระหว่างกระบวนการหรือเซิร์ฟเวอร์ ข้อมูลหายเมื่อกระบวนการรีสตาร์ท มักใช้กับโครงการที่ข้อมูลแคชไม่มาก

#### ไดรเวอร์ apcu
จัดเก็บในหน่วยความจำ ประสิทธิภาพรองจาก array รองรับการแชร์แคชระหว่างกระบวนการ ไม่รองรับการแชร์ระหว่างหลายเซิร์ฟเวอร์ ข้อมูลหายเมื่อกระบวนการรีสตาร์ท มักใช้กับโครงการที่ข้อมูลแคชไม่มาก

> ต้องติดตั้งและเปิดใช้งาน [ส่วนขยาย APCu](https://pecl.php.net/package/APCu) ไม่แนะนำสำหรับสถานการณ์ที่เขียน/ลบแคชบ่อย จะทำให้ประสิทธิภาพลดลงอย่างเห็นได้ชัด

#### ไดรเวอร์ redis
พึ่งพาคอมโพเนนต์ [webman/redis](./redis.md) รองรับการแชร์แคชระหว่างกระบวนการและเซิร์ฟเวอร์

**stores.redis.connection**

`stores.redis.connection` ตรงกับ key ที่กำหนดใน `config/redis.php` เมื่อใช้ Redis จะนำการกำหนดค่าของ `webman/redis` มาใช้รวมถึงการกำหนดค่า connection pool

**แนะนำให้เพิ่มการกำหนดค่า Redis แยกสำหรับแคชใน `config/redis.php` เช่น**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== เพิ่มใหม่
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

จากนั้นตั้ง `stores.redis.connection` เป็น `cache` ไฟล์ `config/cache.php` สุดท้ายจะคล้ายดังนี้:

```php
<?php
return [
    'default' => 'redis', // <====
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache' // <====
        ],
        'array' => [
            'driver' => 'array'
        ]
    ]
];
```

## การเปลี่ยนที่เก็บ
สามารถเปลี่ยนที่เก็บด้วยตนเองเพื่อใช้ไดรเวอร์ที่แตกต่างกัน เช่น:

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **เคล็ดลับ**
> ชื่อ key แคชถูกจำกัดตาม [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) ห้ามมีอักขระใดๆ ใน `{}()/\@:` การตรวจสอบนี้ตั้งแต่ `symfony/cache` 7.2.4 สามารถข้ามชั่วคราวได้ด้วยการกำหนดค่า PHP ini `zend.assertions=-1`

## การใช้คอมโพเนนต์แคชอื่น

ดู [ฐานข้อมูลอื่น ๆ](others.md#ThinkCache) สำหรับคอมโพเนนต์ [ThinkCache](https://github.com/webman-php/think-cache)
