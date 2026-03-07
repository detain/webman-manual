# 이미지 처리 컴포넌트

## 프로젝트 주소

https://github.com/Intervention/image
  
## 설치
 
```php
composer require intervention/image
```
  
## 사용법

**업로드 페이지 스니펫**

```html
  <form method="post" action="/user/img" enctype="multipart/form-data">
      <input type="file" name="file">
      <input type="submit" value="제출">
  </form>
```

**`app/controller/UserController.php` 새로 만들기**

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

> **참고**
> 위 예제는 v3 버전 사용법입니다

## 자세한 내용

https://image.intervention.io/v3 를 방문하세요
  
