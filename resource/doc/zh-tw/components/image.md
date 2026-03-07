# 圖像處理元件

## 專案位址

https://github.com/Intervention/image
  
## 安裝
 
```php
composer require intervention/image
```
  
## 使用

**上傳頁面片段**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="提交">
  </form>
```

**新增 `app/controller/UserController.php`**

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

> **注意**
> 以上為 v3 版本用法

## 更多內容

造訪 https://image.intervention.io/v3
  
