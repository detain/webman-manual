# مكوّن معالجة الصور

## عنوان المشروع

https://github.com/Intervention/image
  
## التثبيت
 
```php
composer require intervention/image
```
  
## الاستخدام

**مقتطف من صفحة الرفع**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="إرسال">
  </form>
```

**إنشاء `app/controller/UserController.php`**

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

> **ملاحظة**
> المثال أعلاه يستخدم واجهة v3.

## المزيد من المعلومات

قم بزيارة https://image.intervention.io/v3
  
