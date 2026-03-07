# متعدد اللغات

يستخدم الدعم متعدد اللغات مكوّن [symfony/translation](https://github.com/symfony/translation).

## التثبيت
```
composer require symfony/translation
```

## إنشاء حزم اللغات
يخزن webman حزم اللغات افتراضياً في المجلد `resource/translations` (أُنشئه إن لم يكن موجوداً). لتغيير المجلد، اضبطه في `config/translation.php`.
كل لغة تتوافق مع مجلد فرعي، وتُخزَّن تعريفات اللغة في `messages.php` افتراضياً. مثال:
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

جميع ملفات اللغات تُرجع مصفوفة، مثلاً:
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## الإعداد

`config/translation.php`

```php
return [
    // اللغة الافتراضية
    'locale' => 'zh_CN',
    // اللغة الاحتياطية: عند عدم إيجاد الترجمة في اللغة الحالية، تُجرّب ترجمة اللغة الاحتياطية
    'fallback_locale' => ['zh_CN', 'en'],
    // مجلد تخزين ملفات اللغات
    'path' => base_path() . '/resource/translations',
];
```

## الترجمة

استخدم الدالة `trans()` للترجمة.

أنشئ ملف اللغة `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 世界!',
];
```

أنشئ الملف `app/controller/UserController.php`:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        $hello = trans('hello'); // 你好 世界!
        return response($hello);
    }
}
```

زيارة `http://127.0.0.1:8787/user/get` ستعيد "你好 世界!"

## تغيير اللغة الافتراضية

استخدم الدالة `locale()` لتبديل اللغة.

أضف ملف اللغة `resource/translations/en/messages.php`:
```php
return [
    'hello' => 'hello world!',
];
```

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // تبديل اللغة
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
زيارة `http://127.0.0.1:8787/user/get` ستعيد "hello world!"

يمكنك أيضاً استخدام المعامل الرابع لدالة `trans()` لتبديل اللغة مؤقتاً. المثال أعلاه يعادل:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // المعامل الرابع يبدّل اللغة
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## تعيين اللغة صراحة لكل طلب
translation كائن واحد (singleton)، أي أن جميع الطلبات تشارك نفس النسخة. إن عيّن طلب اللغة الافتراضية بـ `locale()`، سيؤثر ذلك على جميع الطلبات اللاحقة في العملية. لذلك يجب تعيين اللغة صراحة لكل طلب، مثلاً باستخدام الوسيط التالي:

أنشئ الملف `app/middleware/Lang.php` (أنشئ المجلد إن لم يكن موجوداً):
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Lang implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        locale(session('lang', 'zh_CN'));
        return $handler($request);
    }
}
```

أضف الوسيط العام في `config/middleware.php`:
```php
return [
    // الوسيط العام
    '' => [
        // ... وسيطات أخرى محذوفة
        app\middleware\Lang::class,
    ]
];
```


## استخدام العلامات النائبة
أحياناً تتضمن رسالة متغيرات تحتاج ترجمة، مثلاً
```php
trans('hello ' . $name);
```
في هذه الحالة نستخدم العلامات النائبة.

عدّل `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 %name%!',
];
```
عند الترجمة نمرّر القيم عبر المعامل الثاني:
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## التعامل مع الجمع
في بعض اللغات يختلف تركيب الجملة حسب الكمية. مثلاً "There is %count% apple" صحيح عندما `%count%` يساوي 1، وخاطئ عند أكبر من 1.

في هذه الحالة نستخدم **الخط العمودي** (`|`) لسرد صيغ الجمع.

أضف `apple_count` في ملف `resource/translations/en/messages.php`:
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

يمكننا أيضاً تحديد نطاقات رقمية لإنشاء قواعد جمع أكثر تعقيداً:
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## تحديد ملف اللغة

اسم الملف الافتراضي هو `messages.php`، ويمكنك إنشاء ملفات لغات بأسماء أخرى.

أنشئ ملف `resource/translations/zh_CN/admin.php`:
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

حدّد ملف اللغة بالمعامل الثالث لـ `trans()` (احذف اللاحقة `.php`):
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## مزيد من المعلومات
راجع [توثيق symfony/translation](https://symfony.com/doc/current/translation.html)
