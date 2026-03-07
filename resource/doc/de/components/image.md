# Bildverarbeitungskomponente

## Projektadresse

https://github.com/Intervention/image
  
## Installation
 
```php
composer require intervention/image
```
  
## Verwendung

**Upload-Seitenausschnitt**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="Absenden">
  </form>
```

**Neue Datei `app/controller/UserController.php` erstellen**

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

> **Hinweis**
> Das obige Beispiel verwendet die v3 API.

## Weitere Informationen

Besuchen Sie https://image.intervention.io/v3
  
