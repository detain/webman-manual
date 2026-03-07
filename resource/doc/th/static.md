# webman การจัดการไฟล์สถิติ

webman รองรับการเข้าถึงไฟล์สถิติ โดยไฟล์สถิติจะถูกวางไว้ในไดเร็กทอรี `public` เช่น การเข้าถึง `http://127.0.0.8787/upload/avatar.png` จริงๆ แล้วคือการเข้าถึง `{ไดเร็กทอรีหลักของโปรเจกต์}/public/upload/avatar.png`

> **โปรดทราบ**
> การเข้าถึงไฟล์สถิติที่ขึ้นต้นด้วย `/app/xx/ชื่อไฟล์` จริงๆ แล้วคือการเข้าถึงไดเร็กทอรี `public` ของปลั๊กอินแอปพลิเคชัน กล่าวคือ ไม่รองรับการเข้าถึงไดเร็กทอรีภายใต้ `{ไดเร็กทอรีหลักของโปรเจกต์}/public/app/`
> ดูเพิ่มเติมได้ที่ [ปลั๊กอินแอปพลิเคชัน](./plugin/app.md)

## ปิดการรองรับไฟล์สถิติ

หากไม่ต้องการรองรับไฟล์สถิติ โปรดเปิดไฟล์ `config/static.php` และเปลี่ยนตัวเลือก `enable` เป็น false หลังจากปิดการรองรับแล้ว ทุกการเข้าถึงไฟล์สถิติจะคืนค่า 404

## เปลี่ยนไดเร็กทอรีของไฟล์สถิติ

webman ใช้ไดเร็กทอรี `public` เป็นไดเร็กทอรีไฟล์สถิติโดยค่าเริ่มต้น หากต้องการเปลี่ยน โปรดแก้ไขฟังก์ชัน `public_path()` ใน `support/helpers.php`

## มิดเดิลแวร์ไฟล์สถิติ

webman มาพร้อมกับมิดเดิลแวร์ไฟล์สถิติ ซึ่งตั้งอยู่ที่ `app/middleware/StaticFile.php`
บางครั้งเราอาจต้องการปรับแต่งไฟล์สถิติ เช่น การเพิ่มหัว HTTP ข้ามโดเมน หรือห้ามการเข้าถึงไฟล์ที่ขึ้นต้นด้วยจุด (`.`) สามารถใช้มิดเดิลแวร์นี้ได้

เนื้อหาภายใน `app/middleware/StaticFile.php` คล้ายกับด้านล่าง:
```php
<?php
namespace support\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class StaticFile implements MiddlewareInterface
{
    public function process(Request $request, callable $next) : Response
    {
        // ห้ามการเข้าถึงไฟล์ที่ขึ้นต้นด้วยจุด
        if (strpos($request->path(), '/.') !== false) {
            return response('<h1>403 forbidden</h1>', 403);
        }
        /** @var Response $response */
        $response = $next($request);
        // เพิ่มหัว HTTP ข้ามโดเมน
        /*$response->withHeaders([
            'Access-Control-Allow-Origin'      => '*',
            'Access-Control-Allow-Credentials' => 'true',
        ]);*/
        return $response;
    }
}
```
หากต้องการใช้มิดเดิลแวร์นี้ โปรดเปิดใช้งานในตัวเลือก `middleware` ใน `config/static.php`
