# คิว Stomp

Stomp เป็นโปรโตคอลข้อความแบบข้อความง่าย (สตรีมมิง) ที่ให้รูปแบบการเชื่อมต่อที่ทำงานร่วมกันได้ ทำให้ไคลเอนต์ STOMP สามารถสื่อสารกับโบรกเกอร์ข้อความ STOMP ใดๆ ได้ [workerman/stomp](https://github.com/walkor/stomp) เป็นการ implement ไคลเอนต์ Stomp ใช้หลักในสถานการณ์คิวข้อความ เช่น RabbitMQ, Apollo, ActiveMQ เป็นต้น

## การติดตั้ง
`composer require webman/stomp`

## การกำหนดค่า
ไฟล์กำหนดค่าอยู่ที่ `config/plugin/webman/stomp`

## การส่งข้อความ
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // คิว
        $queue = 'examples';
        // ข้อมูล (เมื่อส่งอาร์เรย์ ต้องทำการ serialize เอง เช่นใช้ json_encode, serialize เป็นต้น)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // ส่งข้อความ
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> เพื่อความเข้ากันได้กับโปรเจกต์อื่น Stomp component ไม่ได้ให้การ serialize และ deserialize อัตโนมัติ หากส่งข้อมูลเป็นอาร์เรย์ ต้อง serialize เอง และ deserialize เองเมื่อบริโภค

## การบริโภคข้อความ
สร้างไฟล์ `app/queue/stomp/MyMailSend.php` (ชื่อคลาสใดก็ได้ ตามมาตรฐาน PSR-4)
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // ชื่อคิว
    public $queue = 'examples';

    // ชื่อการเชื่อมต่อ ตรงกับการเชื่อมต่อใน stomp.php
    public $connection = 'default';

    // เมื่อค่าเป็น client ต้องเรียก $ack_resolver->ack() เพื่อแจ้งเซิร์ฟเวอร์ว่าบริโภคสำเร็จ
    // เมื่อค่าเป็น auto ไม่ต้องเรียก $ack_resolver->ack()
    public $ack = 'auto';

    // บริโภค
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // ถ้าข้อมูลเป็นอาร์เรย์ ต้อง deserialize เอง
        var_export(json_decode($data, true)); // แสดง ['to' => 'tom@gmail.com', 'content' => 'hello']
        // แจ้งเซิร์ฟเวอร์ว่าบริโภคสำเร็จ
        $ack_resolver->ack(); // เมื่อ ack เป็น auto สามารถละเว้นการเรียกนี้ได้
    }
}
```

# เปิดใช้งานโปรโตคอล Stomp ใน RabbitMQ
RabbitMQ ไม่ได้เปิดใช้งานโปรโตคอล Stomp ตามค่าเริ่มต้น ต้องรันคำสั่งต่อไปนี้เพื่อเปิดใช้งาน
```
rabbitmq-plugins enable rabbitmq_stomp
```
หลังเปิดใช้งาน พอร์ต Stomp ค่าเริ่มต้นคือ 61613
