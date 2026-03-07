# การโหลดอัตโนมัติ

## โหลดไฟล์ตามมาตรฐาน PSR-0 ผ่าน Composer
webman ปฏิบัติตามมาตรฐานการโหลดอัตโนมัติ `PSR-4` หากโปรเจกต์ของคุณต้องโหลดไลบรารีที่สอดคล้องกับ PSR-0 ให้ทำตามขั้นตอนต่อไปนี้:

- สร้างไดเร็กทอรี `extend` เพื่อเก็บไลบรารี PSR-0
- แก้ไข `composer.json` และเพิ่มเนื้อหาต่อไปนี้ใน `autoload`:

```json
"psr-0" : {
    "": "extend/"
}
```
ผลลัพธ์สุดท้ายจะคล้ายกับนี้:
![](../../assets/img/psr0.png)

- รันคำสั่ง `composer dumpautoload`
- รันคำสั่ง `php start.php restart` เพื่อรีสตาร์ท webman (หมายเหตุ: ต้องรีสตาร์ทจึงจะมีผล)

## โหลดไฟล์เฉพาะบางไฟล์ผ่าน Composer

- แก้ไข `composer.json` และเพิ่มไฟล์ที่จะโหลดใน `autoload.files`:
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- รันคำสั่ง `composer dumpautoload`
- รันคำสั่ง `php start.php restart` เพื่อรีสตาร์ท webman (หมายเหตุ: ต้องรีสตาร์ทจึงจะมีผล)

> **หมายเหตุ**
> ไฟล์ที่กำหนดใน `autoload.files` ของ composer.json จะถูกโหลดก่อนที่ webman จะเริ่มทำงาน ส่วนไฟล์ที่โหลดผ่าน `config/autoload.php` ของเฟรมเวิร์กจะถูกโหลดหลังจาก webman เริ่มทำงานแล้ว
> การเปลี่ยนแปลงไฟล์ใน `autoload.files` ของ composer.json ต้อง restart จึงจะมีผล reload ไม่มีผล ส่วนไฟล์ที่โหลดผ่าน `config/autoload.php` ของเฟรมเวิร์กรองรับ hot-reload การเปลี่ยนแปลงจะมีผลเมื่อ reload

## โหลดไฟล์เฉพาะบางไฟล์ผ่านเฟรมเวิร์ก
บางไฟล์อาจไม่สอดคล้องกับมาตรฐาน PSR จึงโหลดอัตโนมัติไม่ได้ คุณสามารถโหลดได้โดยกำหนดค่าใน `config/autoload.php` เช่น:
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **หมายเหตุ**
 > ใน `autoload.php` กำหนดให้โหลดไฟล์ `support/Request.php` และ `support/Response.php` เพราะใน `vendor/workerman/webman-framework/src/support/` ก็มีไฟล์ชื่อเดียวกัน ผ่าน `autoload.php` เราให้優先โหลดไฟล์ในไดเร็กทอรีรากของโปรเจกต์ เพื่อให้สามารถปรับแต่งเนื้อหาได้โดยไม่ต้องแก้ไฟล์ใน `vendor` หากไม่ต้องปรับแต่ง สามารถละเว้นการตั้งค่าทั้งสองนี้ได้
