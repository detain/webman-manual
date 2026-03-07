# คอมโพเนนต์การประมวลผลภาพ

## ที่อยู่โปรเจกต์

https://github.com/Intervention/image
  
## การติดตั้ง
 
```php
composer require intervention/image
```
  
## การใช้งาน

**ตัวอย่างโค้ดหน้าแอปโหลด**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="ส่ง">
  </form>
```

**สร้างไฟล์ใหม่ `app/controller/UserController.php`**

```php
<?php
namespace app\controller;
use support\Request;
use Intervention\Image\ImageManager;
use Intervention\Image\Drivers\Gd\Driver;

class UserController
{
    public function img(Request $request)
    {
        $file = $request->file('file');
        if ($file && $file->isValid()) {
            $manager = new ImageManager(new Driver());
            $image = $manager->read($file)->scale(100, 100);
            return response($image->encode(), 200, ['Content-Type' => 'image/png']);
        }
        return response('file not found');
    }
    
}
```

> **หมายเหตุ**
> ตัวอย่างด้านบนใช้ API เวอร์ชัน v3

## ข้อมูลเพิ่มเติม

เยี่ยมชม https://image.intervention.io/v3
  
