# إدارة الجلسة في webman

## مثال
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

احصل على مثيل `Workerman\Protocols\Http\Session` عبر `$request->session();` واستخدم طرقه لإضافة أو تعديل أو حذف بيانات الجلسة.

> **ملاحظة**
> تُحفظ بيانات الجلسة تلقائياً عند إتلاف كائن الجلسة.
> تخزين كائن الجلسة في متغير عام يمنع إتلافه وبالتالي عدم الحفظ التلقائي. في هذه الحالة يجب استدعاء `$session->save()` يدوياً لحفظ البيانات.

## الحصول على جميع بيانات الجلسة
```php
$session = $request->session();
$all = $session->all();
```
تُرجع مصفوفة. إذا لم تكن هناك بيانات جلسة، تُرجع مصفوفة فارغة.


## الحصول على قيمة من الجلسة
```php
$session = $request->session();
$name = $session->get('name');
```
إذا لم تكن البيانات موجودة، تُرجع null.

يمكنك تمرير قيمة افتراضية كمعامل ثاني للطريقة `get`. إذا لم يُعثر على القيمة المقابلة في مصفوفة الجلسة، تُرجع القيمة الافتراضية. مثلاً:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## تخزين بيانات الجلسة
استخدم الطريقة `set` لتخزين قيمة واحدة.
```php
$session = $request->session();
$session->set('name', 'tom');
```
الطريقة `set` لا ترجع شيئاً. تُحفظ بيانات الجلسة تلقائياً عند إتلاف كائن الجلسة.

استخدم الطريقة `put` لتخزين عدة قيم.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
وبالمثل، الطريقة `put` لا ترجع شيئاً.

## حذف بيانات الجلسة
استخدم الطريقة `forget` لحذف قيمة أو أكثر من الجلسة.
```php
$session = $request->session();
// حذف قيمة واحدة
$session->forget('name');
// حذف عدة قيم
$session->forget(['name', 'age']);
```

يتوفر أيضاً الطريقة `delete`. على عكس `forget`، يمكنه حذف قيمة واحدة فقط.
```php
$session = $request->session();
// يعادل $session->forget('name');
$session->delete('name');
```


## الحصول على قيمة من الجلسة وحذفها
```php
$session = $request->session();
$name = $session->pull('name');
```
هذا يعادل الكود التالي:
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
تُرجع null إذا لم تكن قيمة الجلسة المقابلة موجودة.


## حذف جميع بيانات الجلسة
```php
$request->session()->flush();
```
لا ترجع شيئاً. تُحذف بيانات الجلسة تلقائياً من التخزين عند إتلاف كائن الجلسة.


## التحقق من وجود قيمة في الجلسة
```php
$session = $request->session();
$has = $session->has('name');
```
تُرجع false إذا لم تكن القيمة موجودة أو كانت null؛ وإلا تُرجع true.

```php
$session = $request->session();
$has = $session->exists('name');
```
الكود أعلاه يتحقق أيضاً من وجود قيمة في الجلسة. الفرق أن `exists` تُرجع true حتى عندما تكون القيمة null.

## الدالة المساعدة session()

يوفر webman الدالة المساعدة `session()` لنفس الوظائف.
```php
// الحصول على مثيل الجلسة
$session = session();
// يعادل
$session = $request->session();

// الحصول على قيمة
$value = session('key', 'default');
// يعادل
$value = session()->get('key', 'default');
// يعادل
$value = $request->session()->get('key', 'default');

// تعيين قيم للجلسة
session(['key1'=>'value1', 'key2' => 'value2']);
// يعادل
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// يعادل
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## ملف الإعدادات
ملف إعدادات الجلسة في `config/session.php`. المحتوى مشابه لما يلي:
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class أو RedisSessionHandler::class أو RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // إذا كان handler هو FileSessionHandler::class فالقيمة 'file'
    // إذا كان handler هو RedisSessionHandler::class فالقيمة 'redis'
    // إذا كان handler هو RedisClusterSessionHandler::class فالقيمة 'redis_cluster' (عنقود Redis)
    'type'    => 'file',

    // كل معالج يستخدم إعدادات مختلفة
    'config' => [
        // إعدادات عندما يكون type هو 'file'
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // إعدادات عندما يكون type هو 'redis'
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // اسم كوكي تخزين session_id
    'auto_update_timestamp' => false,  // تحديث الجلسة تلقائياً، الافتراضي: معطل
    'lifetime' => 7*24*60*60,          // مدة صلاحية الجلسة
    'cookie_lifetime' => 365*24*60*60, // مدة صلاحية كوكي session_id
    'cookie_path' => '/',              // مسار كوكي session_id
    'domain' => '',                    // نطاق كوكي session_id
    'http_only' => true,               // تفعيل httpOnly، الافتراضي: مفعل
    'secure' => false,                 // تفعيل الجلسة فقط عبر HTTPS، الافتراضي: معطل
    'same_site' => '',                 // لمنع هجمات CSRF وتتبع المستخدمين، القيم: strict/lax/none
    'gc_probability' => [1, 1000],     // احتمال جمع بيانات الجلسة
];
```

## الأمان
لا يُنصح بتخزين مثيلات الكلاسات مباشرة في الجلسة، خاصة من مصادر غير موثوقة. إلغاء التسلسل قد يتسبب في مخاطر أمنية.

