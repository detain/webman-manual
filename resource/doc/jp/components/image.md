# 画像処理コンポーネント

## プロジェクトアドレス

https://github.com/Intervention/image
  
## インストール
 
```php
composer require intervention/image
```
  
## 使用方法

**アップロードページのスニペット**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="送信">
  </form>
```

**`app/controller/UserController.php` を新規作成**

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
> 上記は v3 バージョンの使用方法です

## 詳細情報

https://image.intervention.io/v3 をご覧ください
  
