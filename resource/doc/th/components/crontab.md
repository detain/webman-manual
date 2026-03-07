# คอมโพเนนต์งานตามเวลา crontab

## คำอธิบาย

`workerman/crontab` เหมือนกับ crontab ใน Linux แต่ต่างกันที่รองรับการตั้งเวลาในหน่วยวินาที

รูปแบบเวลา:

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[ละได้ ถ้าไม่มีตำแหน่ง 0 หน่วยเวลาขั้นต่ำคือนาที]
```

## URL โครงการ

https://github.com/walkor/crontab
  
## การติดตั้ง
 
```php
composer require workerman/crontab
```
  
## การใช้งาน

**ขั้นตอนที่ 1: สร้างไฟล์โปรเซส `app/process/Task.php`**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {

        // ทำงานทุกวินาที
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // ทำงานทุก 5 วินาที
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // ทำงานทุกนาที
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // ทำงานทุก 5 นาที
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // ทำงานในวินาทีแรกของทุกนาที
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // ทำงานทุกวันเวลา 7:50 (ตำแหน่งวินาทีละไว้ที่นี่)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**ขั้นตอนที่ 2: กำหนดค่าโปรเซสให้เริ่มพร้อม webman**
  
เปิดไฟล์ `config/process.php` แล้วเพิ่มดังนี้:

```php
return [
    ....ค่าอื่นๆ ละไว้....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**ขั้นตอนที่ 3: รีสตาร์ท webman**

> หมายเหตุ: งานตามเวลาไม่ได้ทำงานทันที งานทั้งหมดจะเริ่มนับและทำงานตั้งแต่นาทีถัดไป

## หมายเหตุ
crontab ไม่ใช่แบบไม่รอ (asynchronous) ตัวอย่าง: ในโปรเซส task มีตัวตั้งเวลา A และ B ทั้งคู่ทำงานทุกวินาที แต่ถ้างาน A ใช้เวลา 10 วินาที งาน B ต้องรอ A เสร็จก่อนจึงจะทำงาน ทำให้ B ทำงานล่าช้า
ถ้าตรรกะต้องการความแม่นยำของช่วงเวลา ให้ทำงานตามเวลาที่ต้องการความแม่นยำในโปรเซสแยก เพื่อไม่ให้งานอื่นรบกวน ตัวอย่างการตั้งค่า `config/process.php`:

```php
return [
    ....ค่าอื่นๆ ละไว้....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
วางงานตามเวลาที่ต้องการความแม่นยำใน `process/Task1.php` งานอื่นใน `process/Task2.php`

รายละเอียดเพิ่มเติมของ `config/process.php` ดูที่ [โปรเซสที่กำหนดเอง](../process.md)
