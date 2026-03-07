# Компонент обработки изображений

## Адрес проекта

https://github.com/Intervention/image
  
## Установка
 
```php
composer require intervention/image
```
  
## Использование

**Фрагмент страницы загрузки**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="Отправить">
  </form>
```

**Создать `app/controller/UserController.php`**

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

> **Примечание**
> Приведённый выше пример использует API v3.

## Подробнее

Посетите https://image.intervention.io/v3
  
