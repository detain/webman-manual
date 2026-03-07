# Thành phần xử lý ảnh

## Địa chỉ dự án

https://github.com/Intervention/image
  
## Cài đặt
 
```php
composer require intervention/image
```
  
## Sử dụng

**Đoạn mã trang tải lên**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="Gửi">
  </form>
```

**Tạo mới `app/controller/UserController.php`**

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

> **Lưu ý**
> Ví dụ trên sử dụng API v3.

## Thông tin thêm

Truy cập https://image.intervention.io/v3
  
