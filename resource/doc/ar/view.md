# العرض
يستخدم webman بشكل افتراضي بناء جملة PHP الأصلي كقالب، ويحقق أفضل أداء عند تفعيل `opcache`. بالإضافة إلى قالب PHP الأصلي، يوفر webman محركات قوالب [Twig](https://twig.symfony.com/doc/3.x/)، [Blade](https://learnku.com/docs/laravel/8.x/blade/9377)، [think-template](https://www.kancloud.cn/manual/think-template/content).

## تفعيل opcache
عند استخدام العرض، يُوصى بشدة بتفعيل خياري `opcache.enable` و `opcache.enable_cli` في ملف php.ini لتحقيق أفضل أداء لمحرك القوالب.

## تثبيت Twig
1. التثبيت عبر composer

   `composer require twig/twig`

2. تعديل الإعدادات في `config/view.php` كالتالي
   ```php
   <?php
   use support\view\Twig;

   return [
       'handler' => Twig::class
   ];
   ```
   > **ملاحظة**
   > يمكن تمرير خيارات الإعدادات الأخرى عبر options، مثلاً:

   ```php
   return [
       'handler' => Twig::class,
       'options' => [
           'debug' => false,
           'charset' => 'utf-8'
       ]
   ];
   ```

## تثبيت Blade
1. التثبيت عبر composer

   ```
   composer require psr/container ^1.1.1 webman/blade
   ```

2. تعديل الإعدادات في `config/view.php` كالتالي
   ```php
   <?php
   use support\view\Blade;

   return [
       'handler' => Blade::class
   ];
   ```

## تثبيت think-template
1. التثبيت عبر composer

   `composer require topthink/think-template`

2. تعديل الإعدادات في `config/view.php` كالتالي
   ```php
   <?php
   use support\view\ThinkPHP;

   return [
       'handler' => ThinkPHP::class,
   ];
   ```
   > **ملاحظة**
   > يمكن تمرير خيارات الإعدادات الأخرى عبر options، مثلاً:

   ```php
   return [
       'handler' => ThinkPHP::class,
       'options' => [
           'view_suffix' => 'html',
           'tpl_begin' => '{',
           'tpl_end' => '}'
       ]
   ];
   ```

## مثال على محرك قالب PHP الأصلي
إنشاء الملف `app/controller/UserController.php` كالتالي:

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        return view('user/hello', ['name' => 'webman']);
    }
}
```

إنشاء الملف `app/view/user/hello.html` كالتالي:

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>webman</title>
</head>
<body>
hello <?=htmlspecialchars($name)?>
</body>
</html>
```

## مثال على محرك قالب Twig

تعديل الإعدادات في `config/view.php` كالتالي:
```php
<?php
use support\view\Twig;

return [
    'handler' => Twig::class
];
```

الملف `app/controller/UserController.php` كالتالي:

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        return view('user/hello', ['name' => 'webman']);
    }
}
```

الملف `app/view/user/hello.html` كالتالي:

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>webman</title>
</head>
<body>
hello {{name}}
</body>
</html>
```

للمزيد من الوثائق راجع [Twig](https://twig.symfony.com/doc/3.x/)

## مثال على قالب Blade
تعديل الإعدادات في `config/view.php` كالتالي:
```php
<?php
use support\view\Blade;

return [
    'handler' => Blade::class
];
```

الملف `app/controller/UserController.php` كالتالي:

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        return view('user/hello', ['name' => 'webman']);
    }
}
```

الملف `app/view/user/hello.blade.php` كالتالي:

> لاحظ أن امتداد قالب blade هو `.blade.php`

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>webman</title>
</head>
<body>
hello {{$name}}
</body>
</html>
```

للمزيد من الوثائق راجع [Blade](https://learnku.com/docs/laravel/8.x/blade/9377)

## مثال على قالب ThinkPHP
تعديل الإعدادات في `config/view.php` كالتالي:
```php
<?php
use support\view\ThinkPHP;

return [
    'handler' => ThinkPHP::class
];
```

الملف `app/controller/UserController.php` كالتالي:

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        return view('user/hello', ['name' => 'webman']);
    }
}
```

الملف `app/view/user/hello.html` كالتالي:

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>webman</title>
</head>
<body>
hello {$name}
</body>
</html>
```

للمزيد من الوثائق راجع [think-template](https://www.kancloud.cn/manual/think-template/content)

## تعيين القالب
بالإضافة إلى استخدام `view(القالب، مصفوفة_المتغيرات)` لتعيين القالب، يمكننا أيضاً استدعاء `View::assign()` في أي مكان لتعيين القالب. مثلاً:
```php
<?php
namespace app\controller;

use support\Request;
use support\View;

class UserController
{
    public function hello(Request $request)
    {
        View::assign([
            'name1' => 'value1',
            'name2'=> 'value2',
        ]);
        View::assign('name3', 'value3');
        return view('user/test', ['name' => 'webman']);
    }
}
```

يكون `View::assign()` مفيداً جداً في بعض السيناريوهات، مثلاً عندما يتعين عرض معلومات المستخدم الحالي المسجل دخوله في رأس كل صفحة في نظام ما. سيكون من المرهق تعيين هذه المعلومات في كل صفحة عبر `view('القالب', ['user_info' => 'معلومات المستخدم']);`. الحل هو الحصول على معلومات المستخدم في الوسيط ثم تعيينها للقالب عبر `View::assign()`.

## حول مسار ملف العرض

#### المتحكم
عندما يستدعي المتحكم `view('اسم_القالب',[]);`، يتم البحث عن ملف العرض وفق القواعد التالية:

1. إذا بدأ المسار بـ `/`، استخدم هذا المسار مباشرة للبحث عن ملف العرض
2. إذا لم يبدأ بـ `/` ولم تكن تطبيقاً متعدداً، استخدم ملف العرض المقابل تحت `app/view/`
3. إذا لم يبدأ بـ `/` وكان [تطبيقاً متعدداً](multiapp.md)، استخدم ملف العرض المقابل تحت `app/اسم_التطبيق/view/`
4. إذا لم يتم تمرير معامل القالب، يتم البحث عن ملف القالب تلقائياً وفق القاعدتين 2 و 3

مثال:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        // يعادل return view('user/hello', ['name' => 'webman']);
        // يعادل return view('/app/view/user/hello', ['name' => 'webman']);
        return view(['name' => 'webman']);
    }
}
```

#### دالة الإغلاق
دالة الإغلاق `$request->app` فارغة ولا تنتمي إلى أي تطبيق، لذلك تستخدم دالة الإغلاق ملفات العرض تحت `app/view/`، مثلاً عند تعريف المسار في `config/route.php`:
```php
Route::any('/admin/user/get', function (Request $request) {
    return view('user', []);
});
```
سيتم استخدام `app/view/user.html` كملف قالب (عند استخدام قالب blade يكون ملف القالب `app/view/user.blade.php`).

#### تحديد التطبيق
لتمكين إعادة استخدام القوالب في وضع التطبيقات المتعددة، يوفر `view($template, $data, $app = null)` المعامل الثالث `$app` لتحديد قالب أي مجلد تطبيق سيتم استخدامه. مثلاً `view('user', [], 'admin');` سيُجبر على استخدام ملفات العرض تحت `app/admin/view/`.

#### حذف معامل القالب
في المتحكمات المعتمدة على الصنف، يمكنك حذف معامل القالب. مثلاً:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        // يعادل return view('user/hello', ['name' => 'webman']);
        // يعادل return view('/app/view/user/hello', ['name' => 'webman']);
        return view(['name' => 'webman']);
    }
}
```

## تمديد twig

يمكننا تمديد مثيل عرض twig من خلال توفير رد اتصال `view.extension` في الإعدادات، مثلاً `config/view.php` كالتالي:
```php
<?php
use support\view\Twig;
return [
    'handler' => Twig::class,
    'extension' => function (\Twig\Environment $twig) {
        $twig->addExtension(new your\namespace\YourExtension()); // إضافة امتداد
        $twig->addFilter(new \Twig\TwigFilter('rot13', 'str_rot13')); // إضافة فلتر
        $twig->addFunction(new \Twig\TwigFunction('function_name', function () {})); // إضافة دالة
    }
];
```

## تمديد blade

وبالمثل يمكننا تمديد مثيل عرض blade من خلال توفير رد اتصال `view.extension` في الإعدادات، مثلاً `config/view.php` كالتالي:

```php
<?php
use support\view\Blade;
return [
    'handler' => Blade::class,
    'extension' => function (Jenssegers\Blade\Blade $blade) {
        // إضافة توجيهات إلى blade
        $blade->directive('mydate', function ($timestamp) {
            return "<?php echo date('Y-m-d H:i:s', $timestamp); ?>";
        });
    }
];
```

## استخدام مكون component في blade

لنفترض أننا بحاجة إلى إضافة مكون Alert

**إنشاء `app/view/components/Alert.php` جديد**
```php
<?php

namespace app\view\components;

use Illuminate\View\Component;

class Alert extends Component
{
    
    public function __construct()
    {
    
    }
    
    public function render()
    {
        return view('components/alert')->rawBody();
    }
}
```

**إنشاء `app/view/components/alert.blade.php` جديد**
```
<div>
    <b style="color: red">hello blade component</b>
</div>
```

**الكود في `/config/view.php` مشابه لما يلي:**

```php
<?php
use support\view\Blade;
return [
    'handler' => Blade::class,
    'extension' => function (Jenssegers\Blade\Blade $blade) {
        $blade->component('alert', app\view\components\Alert::class);
    }
];
```

بهذا يكون مكون Blade Alert جاهزاً، وعند الاستخدام في القالب يكون كما يلي:
```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>webman</title>
</head>
<body>

<x-alert/>

</body>
</html>
```

## تمديد think-template
يستخدم think-template `view.options.taglib_pre_load` لتمديد مكتبة العلامات، مثلاً:
```php
<?php
use support\view\ThinkPHP;
return [
    'handler' => ThinkPHP::class,
    'options' => [
        'taglib_pre_load' => your\namspace\Taglib::class,
    ]
];
```

للتفاصيل راجع [تمديد علامات think-template](https://www.kancloud.cn/manual/think-template/1286424)
