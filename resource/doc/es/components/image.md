# Componente de procesamiento de imágenes

## Dirección del proyecto

https://github.com/Intervention/image
  
## Instalación
 
```php
composer require intervention/image
```
  
## Uso

**Fragmento de página de carga**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="Enviar">
  </form>
```

**Crear `app/controller/UserController.php`**

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

> **Nota**
> El ejemplo anterior utiliza la API v3.

## Más información

Visite https://image.intervention.io/v3
  
