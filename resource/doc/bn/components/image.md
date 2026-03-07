# চিত্র প্রসেসিং কম্পোনেন্ট

## প্রকল্পের ঠিকানা

https://github.com/Intervention/image
  
## ইনস্টলেশন
 
```php
composer require intervention/image
```
  
## ব্যবহার

**আপলোড পেজের খণ্ড**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="জমা দিন">
  </form>
```

**নতুন `app/controller/UserController.php` তৈরি করুন**

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

> **দ্রষ্টব্য**
> উপরের উদাহরণ v3 API ব্যবহার করে।

## আরও তথ্য

https://image.intervention.io/v3 ভিজিট করুন
  
