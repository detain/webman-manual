# Görüntü İşleme Bileşeni

## Proje Adresi

https://github.com/Intervention/image
  
## Kurulum
 
```php
composer require intervention/image
```
  
## Kullanım

**Yükleme sayfası parçası**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="Gönder">
  </form>
```

**`app/controller/UserController.php` dosyasını oluşturun**

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

> **Not**
> Yukarıdaki örnek v3 API kullanmaktadır.

## Daha fazla bilgi

https://image.intervention.io/v3 adresini ziyaret edin
  
