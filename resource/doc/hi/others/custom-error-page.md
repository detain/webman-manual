# कस्टम 404

यदि आप 404 की सामग्री को गतिशील रूप से नियंत्रित करना चाहते हैं, उदाहरण के लिए AJAX अनुरोध में JSON डेटा `{"code:"404", "msg":"404 not found"}` वापस करना और पेज अनुरोध में `app/view/404.html` टेम्पलेट वापस करना, तो निम्नलिखित उदाहरण देखें।

> नीचे PHP मूल टेम्पलेट का उदाहरण है। अन्य टेम्पलेट जैसे `twig`, `blade`, `think-template` भी इसी सिद्धांत का पालन करते हैं।

**`app/view/404.html` फ़ाइल बनाएं**
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

**`config/route.php` में निम्नलिखित कोड जोड़ें:**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // AJAX अनुरोध में JSON वापस करें
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // पेज अनुरोध में 404.html टेम्पलेट वापस करें
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# कस्टम 405

webman-framework 1.5.23 से, फॉलबैक कॉलबैक `status` पैरामीटर का समर्थन करता है। 404 का मतलब अनुरोध मौजूद नहीं है; 405 का मतलब वर्तमान अनुरोध विधि समर्थित नहीं है (उदाहरण: `Route::post()` से परिभाषित रूट को GET से एक्सेस करना)।

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

# कस्टम 500

**`app/view/500.html` बनाएं**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
कस्टम त्रुटि टेम्पलेट:
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**`app/exception/Handler.php` बनाएं** (निर्देशिका न हो तो स्वयं बनाएं)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * प्रतिक्रिया रेंडर करके वापस करें
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // AJAX अनुरोध में JSON डेटा वापस करें
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // पेज अनुरोध में 500.html टेम्पलेट वापस करें
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**`config/exception.php` कॉन्फ़िगर करें**
```php
return [
    '' => \app\exception\Handler::class,
];
```
