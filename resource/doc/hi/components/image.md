# छवि प्रसंस्करण संबंधी घटक

## परियोजना का पता

https://github.com/Intervention/image
  
## स्थापना
 
```php
composer require intervention/image
```
  
## उपयोग

**अपलोड पृष्ठ अंश**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="सबमिट करें">
  </form>
```

**नया `app/controller/UserController.php` बनाएँ**

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

> **ध्यान दें**
> उपरोक्त उदाहरण v3 API का उपयोग करता है।

## अधिक जानकारी

https://image.intervention.io/v3 पर जाएं
  
