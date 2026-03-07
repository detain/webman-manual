# معالجة الملفات الثابتة في webman

يدعم webman الوصول إلى الملفات الثابتة، وتُوضع جميع الملفات الثابتة في المجلد `public`. على سبيل المثال، الوصول إلى `http://127.0.0.8787/upload/avatar.png` يُعد في الواقع وصولاً إلى `{الدليل الرئيسي للمشروع}/public/upload/avatar.png`.

> **ملاحظة**
> الوصول إلى الملفات الثابتة الذي يبدأ بـ`/app/xx/اسم_الملف` يُعد في الواقع وصولاً إلى مجلد `public` الخاص بملحق التطبيق. بمعنى آخر، لا يُدعم الوصول إلى المجلدات الواقعة تحت `{الدليل الرئيسي للمشروع}/public/app/`.
> لمزيد من المعلومات، راجع [ملحقات التطبيق](./plugin/app.md).

## إيقاف دعم الملفات الثابتة

إذا لم تكن هناك حاجة لدعم الملفات الثابتة، افتح `config/static.php` وغيِّر الخيار `enable` إلى false. بعد الإيقاف، سيُرجع جميع محاولات الوصول إلى الملفات الثابتة خطأ 404.

## تغيير مجلد الملفات الثابتة

يُستخدم webman افتراضياً المجلد `public` كمجلد للملفات الثابتة. للتعديل، عدّل الدالة المساعدة `public_path()` في الملف `support/helpers.php`.

## وسيط الملفات الثابتة

يتضمّن webman وسيطاً للملفات الثابتة موجوداً في `app/middleware/StaticFile.php`.
أحياناً نحتاج إلى معالجة الملفات الثابتة، مثل إضافة رؤوس HTTP للسماح بالوصول العابر للنطاقات، أو منع الوصول إلى الملفات التي تبدأ بنقطة (`.`)، يمكن استخدام هذا الوسيط.

محتويات `app/middleware/StaticFile.php` شبيهة بما يلي:
```php
<?php
namespace support\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class StaticFile implements MiddlewareInterface
{
    public function process(Request $request, callable $next) : Response
    {
        // منع الوصول إلى الملفات المخفية التي تبدأ بنقطة
        if (strpos($request->path(), '/.') !== false) {
            return response('<h1>403 forbidden</h1>', 403);
        }
        /** @var Response $response */
        $response = $next($request);
        // إضافة رؤوس الوصول العابر للنطاقات
        /*$response->withHeaders([
            'Access-Control-Allow-Origin'      => '*',
            'Access-Control-Allow-Credentials' => 'true',
        ]);*/
        return $response;
    }
}
```
إذا تطلّب الأمر استخدام هذا الوسيط، يجب تفعيله في خيار `middleware` في ملف `config/static.php`.
