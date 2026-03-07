# การดำเนินงานธุรกิจช้า

บางครั้งเราต้องดำเนินการธุรกิจช้า เพื่อหลีกเลี่ยงผลกระทบต่อการดำเนินการขออื่นใน webman ธุรกิจเหล่านี้สามารถใช้วิธีแก้ไขการดำเนินการที่แตกต่างกันตามสถานการณ์

## วิธีที่ 1: ใช้คิวข้อความ
อ้างอิง [คิว Redis](../queue/redis.md) [คิว Stomp](../queue/stomp.md)

#### ข้อดี
สามารถรับมือกับการร้องขอการดำเนินงานธุรกิจที่เพิ่มขึ้นอย่างกะทันหันได้

#### ข้อเสีย
ไม่สามารถส่งผลลัพธ์โดยตรงไปยังลูกค้า หากต้องการส่งผลลัพธ์จำเป็นต้องประสานงานกับบริการอื่น เช่น ใช้ [webman/push](https://www.workerman.net/plugin/2) ส่งผลการดำเนินงาน

## วิธีที่ 2: เพิ่มพอร์ต HTTP ใหม่

เพิ่มพอร์ต HTTP ใหม่เพื่อจัดการการร้องขอช้า การร้องขอช้าเหล่านี้จะเข้าสู่กลุ่มกระบวนการเฉพาะผ่านพอร์ตนี้ และจะคืนผลลัพธ์โดยตรงให้กับลูกค้าหลังการดำเนินงาน

#### ข้อดี
สามารถส่งข้อมูลโดยตรงไปยังลูกค้าได้

#### ข้อเสีย
ไม่สามารถรับมือกับการร้องขอที่เพิ่มขึ้นอย่างกะทันหันได้

#### ขั้นตอนดำเนินการ
เพิ่มการกำหนดค่าต่อไปนี้ใน `config/process.php`
```php
return [
    // ... ข้ามการกำหนดค่าอื่น ๆ ที่นี่ ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // จำนวนกระบวนการ
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // การกำหนดคลาสคำขอ
            'logger' => \support\Log::channel('default'), // อินสแตนซ์ล็อก
            'appPath' => app_path(), // ตำแหน่งไดเรกทอรี app
            'publicPath' => public_path() // ตำแหน่งไดเรกทอรี public
        ]
    ]
];
```

ดังนั้น อินเทอร์เฟซช้าสามารถทำงานผ่านกลุ่มกระบวนการที่ `http://127.0.0.1:8686/` โดยไม่มีผลกระทบต่อการดำเนินงานธุรกิจของกระบวนการอื่น ๆ

เพื่อให้ลูกค้าไม่รู้สึกถึงความแตกต่างของพอร์ต สามารถเพิ่มพร็อกซีไปยังพอร์ต 8686 ใน nginx ได้ สมมติว่าเส้นทางการร้องขอของอินเทอร์เฟซช้าทั้งหมดเริ่มต้นด้วย `/task` การกำหนดค่า nginx ทั้งหมดจะเป็นดังนี้:
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# เพิ่ม upstream 8686 ใหม่
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # การร้องขอที่เริ่มต้นด้วย /task ไปที่พอร์ต 8686 กรุณาเปลี่ยน /task ตามความต้องการ
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # การร้องขออื่น ๆ ไปที่พอร์ตเดิม 8787
  location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      if (!-f $request_filename){
          proxy_pass http://webman;
      }
  }
}
```

ดังนั้นเมื่อลูกค้าเข้าถึง `domain.com/task/xxx` จะถูกดำเนินการผ่านพอร์ต 8686 แยกต่างหาก โดยไม่มีผลกระทบต่อการดำเนินงานของพอร์ต 8787

## วิธีที่ 3: ใช้ HTTP Chunked ส่งข้อมูลแบบแบ่งส่วนแบบอะซิงโครนัส

#### ข้อดี
สามารถส่งข้อมูลโดยตรงไปยังลูกค้าได้

**ติดตั้ง workerman/http-client**

```
composer require workerman/http-client
```

**app/controller/IndexController.php**
```php
<?php
namespace app\controller;

use support\Request;
use support\Response;
use Workerman\Protocols\Http\Chunk;

class IndexController
{
    public function index(Request $request)
    {
        $connection = $request->connection;
        $http = new \Workerman\Http\Client();
        $http->get('https://example.com/', function ($response) use ($connection) {
            $connection->send(new Chunk($response->getBody()));
            $connection->send(new Chunk('')); // ส่ง chunk ว่างเพื่อแสดงการสิ้นสุดการตอบกลับ
        });
        // ส่งส่วนหัว HTTP ก่อน ข้อมูลถัดไปจะถูกส่งแบบอะซิงโครนัส
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **หมายเหตุ**
> ตัวอย่างนี้ใช้ไคลเอ็นต์ `workerman/http-client` เพื่อดึงผลลัพธ์ HTTP แบบอะซิงโครนัสและส่งกลับข้อมูล คุณยังสามารถใช้ไคลเอ็นต์อะซิงโครนัสอื่นได้ เช่น [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html)
