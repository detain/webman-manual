# Componente de Processamento de Imagens

## Endereço do Projeto

https://github.com/Intervention/image
  
## Instalação
 
```php
composer require intervention/image
```
  
## Utilização

**Trecho da página de upload**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="Enviar">
  </form>
```

**Criar `app/controller/UserController.php`**

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

> **Observação**
> O exemplo acima utiliza a API v3.

## Mais informações

Acesse https://image.intervention.io/v3
  
