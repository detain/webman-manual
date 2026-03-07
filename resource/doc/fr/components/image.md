# Composant de traitement d'image

## Adresse du projet

https://github.com/Intervention/image
  
## Installation
 
```php
composer require intervention/image
```
  
## Utilisation

**Extrait de page de téléchargement**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="Envoyer">
  </form>
```

**Créer `app/controller/UserController.php`**

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

> **Remarque**
> L'exemple ci-dessus utilise l'API v3.

## Plus d'informations

Consultez https://image.intervention.io/v3
  
