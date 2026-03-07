# مثال بسيط للبدء السريع بـ webman

## إرجاع نص
**إنشاء وحدة تحكم جديدة**

أنشئ الملف `app/controller/UserController.php` كما يلي

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $default_name = 'webman';
        // الحصول على المعلمة name من طلب GET، وإرجاع $default_name إذا لم يتم تمرير name
        $name = $request->get('name', $default_name);
        // إرجاع النص إلى المتصفح
        return response('hello ' . $name);
    }
}
```

**الوصول**

افتح `http://127.0.0.1:8787/user/hello?name=tom` في المتصفح

ستعرض المتصفح `hello tom`

## إرجاع JSON
غيّر الملف `app/controller/UserController.php` كما يلي

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $default_name = 'webman';
        $name = $request->get('name', $default_name);
        return json([
            'code' => 0, 
            'msg' => 'ok', 
            'data' => $name
        ]);
    }
}
```

**الوصول**

افتح `http://127.0.0.1:8787/user/hello?name=tom` في المتصفح

ستعرض المتصفح `{"code":0,"msg":"ok","data":"tom"}`

استخدام الدالة المساعدة json لإرجاع البيانات سيضيف تلقائياً رأس `Content-Type: application/json`

## إرجاع XML
وبالمثل، استخدام الدالة المساعدة `xml($xml)` سيعيد استجابة `xml` مع رأس `Content-Type: text/xml`.

يمكن أن تكون المعلمة `$xml` إما سلسلة `xml` أو كائن `SimpleXMLElement`

## إرجاع JSONP
وبالمثل، استخدام الدالة المساعدة `jsonp($data, $callback_name = 'callback')` سيعيد استجابة `jsonp`.

## إرجاع واجهة عرض
غيّر الملف `app/controller/UserController.php` كما يلي

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $default_name = 'webman';
        $name = $request->get('name', $default_name);
        return view('user/hello', ['name' => $name]);
    }
}
```

أنشئ الملف `app/view/user/hello.html` كما يلي

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

افتح `http://127.0.0.1:8787/user/hello?name=tom` في المتصفح
ستتم إعادة صفحة HTML بمحتوى `hello tom`.

ملاحظة: يستخدم webman افتراضياً بناء جملة PHP الأصلي كقالب. لاستخدام واجهات عرض أخرى، راجع [الواجهات](view.md).
