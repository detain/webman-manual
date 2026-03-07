# معالجة الاستثناءات

## التكوين
`config/exception.php`
```php
return [
    // قم بتكوين فئة معالجة الاستثناءات هنا
    '' => support\exception\Handler::class,
];
```
في وضع التطبيقات المتعددة، يمكنك تكوين فئة معالجة الاستثناءات لكل تطبيق بشكل منفصل، راجع [التطبيقات المتعددة](multiapp.md)


## فئة معالجة الاستثناءات الافتراضية
في webman، يتم معالجة الاستثناءات افتراضياً بواسطة فئة `support\exception\Handler`. يمكنك تعديل ملف التكوين `config/exception.php` لتغيير فئة معالجة الاستثناءات الافتراضية. يجب أن تنفذ فئة معالجة الاستثناءات واجهة `Webman\Exception\ExceptionHandlerInterface`.
```php
interface ExceptionHandlerInterface
{
    /**
     * تسجيل السجل
     * @param Throwable $e
     * @return mixed
     */
    public function report(Throwable $e);

    /**
     * عرض الاستجابة
     * @param Request $request
     * @param Throwable $e
     * @return Response
     */
    public function render(Request $request, Throwable $e) : Response;
}
```



## عرض الاستجابة
تُستخدم الطريقة `render` في فئة معالجة الاستثناءات لعرض الاستجابة.

إذا كانت قيمة `debug` في ملف التكوين `config/app.php` هي `true` (يشار إليها فيما يلي بـ `app.debug=true`)، سيتم إرجاع معلومات الاستثناء التفصيلية، وإلا سيتم إرجاع معلومات الاستثناء المختصرة.

إذا كان الطلب يتوقع إرجاع json، فستُرجع معلومات الاستثناء بتنسيق json، مثل
```json
{
    "code": "500",
    "msg": "معلومات الاستثناء"
}
```
إذا كان `app.debug=true`، ستضاف حقل `trace` إضافي في بيانات json لإرجاع تفاصيل مكدس الاستدعاءات.

يمكنك كتابة فئة معالجة استثناءات خاصة بك لتغيير منطق معالجة الاستثناءات الافتراضي.

# استثناء الأعمال BusinessException
أحياناً نريد إنهاء الطلب داخل دالة متداخلة وإرجاع رسالة خطأ للعميل، ويمكن تحقيق ذلك عبر رمي `BusinessException`.
مثال:

```php
<?php
namespace app\controller;

use support\Request;
use support\exception\BusinessException;

class FooController
{
    public function index(Request $request)
    {
        $this->checkInput($request->post());
        return response('hello index');
    }
    
    protected function checkInput($input)
    {
        if (!isset($input['token'])) {
            throw new BusinessException('خطأ في المعلمة', 3000);
        }
    }
}
```

سيُرجع المثال أعلاه
```json
{"code": 3000, "msg": "خطأ في المعلمة"}
```

> **ملاحظة**
> لا يحتاج استثناء الأعمال BusinessException إلى التقاطه بواسطة try في الأعمال، حيث سيقوم الإطار بالتقاطه تلقائياً وإرجاع المخرجات المناسبة وفقاً لنوع الطلب.

## استثناء الأعمال المخصص

إذا لم تكن الاستجابة أعلاه تلبي متطلباتك، على سبيل المثال إذا أردت تغيير `msg` إلى `message`، يمكنك إنشاء استثناء مخصص `MyBusinessException`

أنشئ ملفاً جديداً `app/exception/MyBusinessException.php` بالمحتوى التالي
```php
<?php

namespace app\exception;

use support\exception\BusinessException;
use Webman\Http\Request;
use Webman\Http\Response;

class MyBusinessException extends BusinessException
{
    public function render(Request $request): ?Response
    {
        // طلبات json تُرجع بيانات json
        if ($request->expectsJson()) {
            return json(['code' => $this->getCode() ?: 500, 'message' => $this->getMessage()]);
        }
        // طلبات غير json تُرجع صفحة
        return new Response(200, [], $this->getMessage());
    }
}
```

عند استدعاء الأعمال بهذا الشكل
```php
use app\exception\MyBusinessException;

throw new MyBusinessException('خطأ في المعلمة', 3000);
```
ستستلم طلبات json إرجاع json مشابهاً لما يلي
```json
{"code": 3000, "message": "خطأ في المعلمة"}
```

> **تلميح**
> بما أن استثناء BusinessException ينتمي إلى استثناءات الأعمال (مثل خطأ في معلمات إدخال المستخدم)، فهو متوقع، لذا لن يعتبره الإطار خطأً قاتلاً ولن يسجله في السجل.

## الخلاصة
في أي وقت تريد فيه إنهاء الطلب الحالي وإرجاع معلومات للعميل، يمكنك النظر في استخدام استثناء `BusinessException`.
