# تخصيص 404

إذا أردت التحكم ديناميكياً في محتوى 404، مثلاً إرجاع بيانات JSON `{"code:"404", "msg":"404 not found"}` لطلبات AJAX وإرجاع قالب `app/view/404.html` لطلبات الصفحة، يرجى مراجعة المثال التالي.

> يستخدم المثال التالي قوالب PHP الأصلية، وقوالب أخرى مثل `twig` و`blade` و`think-template` تعمل بنفس المبدأ.

**إنشاء الملف `app/view/404.html`**
```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>404 not found</title>
</head>
<body>
<?=htmlspecialchars($error)?>
</body>
</html>
```

**إضافة الكود التالي في `config/route.php`:**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // إرجاع JSON لطلبات AJAX
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // إرجاع قالب 404.html لطلبات الصفحة
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# تخصيص 405

من إصدار webman-framework 1.5.23، يدعم رد الدالة المعيّن للفشل معامل `status`. القيمة 404 تعني أن الطلب غير موجود، و405 تعني عدم دعم طريقة الطلب الحالية (مثل الوصول بطريقة GET لمسار معرّف بـ `Route::post()`).

```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request, $status) {
    $map = [
        404 => '404 not found',
        405 => '405 method not allowed',
    ];
    return response($map[$status], $status);
});
```

# تخصيص 500

**إنشاء `app/view/500.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
قالب الخطأ المخصص:
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**إنشاء `app/exception/Handler.php`** (أنشئ المجلد إن لم يكن موجوداً)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * عرض الاستجابة وإرجاعها
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // إرجاع بيانات JSON لطلبات AJAX
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // إرجاع قالب 500.html لطلبات الصفحة
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**تكوين `config/exception.php`**
```php
return [
    '' => \app\exception\Handler::class,
];
```
