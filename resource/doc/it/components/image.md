# Componente di elaborazione delle immagini

## Indirizzo del progetto

https://github.com/Intervention/image
  
## Installazione
 
```php
composer require intervention/image
```
  
## Utilizzo

**Snippet della pagina di caricamento**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="Invia">
  </form>
```

**Creare `app/controller/UserController.php`**

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
> L'esempio sopra utilizza l'API v3.

## Ulteriori informazioni

Visita https://image.intervention.io/v3
  
