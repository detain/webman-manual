# المتحكم

وفقًا لمواصفات PSR4، يبدأ مساحة الاسم لفئة المتحكم بـ `plugin\{معرف_الإضافة}`، مثلًا

أنشئ ملف المتحكم الجديد `plugin/foo/app/controller/FooController.php`.

```php
<?php
namespace plugin\foo\app\controller;

use support\Request;

class FooController
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

عند الوصول إلى `http://127.0.0.1:8787/app/foo/foo`، تُرجع الصفحة `hello index`

عند الوصول إلى `http://127.0.0.1:8787/app/foo/foo/hello`، تُرجع الصفحة `hello webman`


## الوصول عبر URL
تبدأ مسارات عناوين URL لإضافة التطبيق بـ `/app`، يليها معرف الإضافة، ثم المتحكم والطريقة المحددة.
مثلًا عنوان URL لـ `plugin\foo\app\controller\UserController` هو `http://127.0.0.1:8787/app/foo/user`
