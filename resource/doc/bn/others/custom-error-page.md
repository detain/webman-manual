# কাস্টম 404

আপনি যদি 404 এর কন্টেন্ট গতিশীলভাবে নিয়ন্ত্রণ করতে চান, উদাহরণস্বরূপ ajax রিকোয়েস্টে JSON ডেটা `{"code:"404", "msg":"404 not found"}` রিটার্ন করা এবং পেজ রিকোয়েস্টে `app/view/404.html` টেমপ্লেট রিটার্ন করা, তাহলে নিচের উদাহরণটি দেখুন।

> নিচে PHP নেটিভ টেমপ্লেটের উদাহরণ দেওয়া হয়েছে। অন্যান্য টেমপ্লেট যেমন `twig`, `blade`, `think-template` একই নীতি অনুসরণ করে।

**`app/view/404.html` ফাইল তৈরি করুন**
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

**`config/route.php` এ নিচের কোড যোগ করুন:**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // ajax রিকোয়েস্টে JSON রিটার্ন করুন
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // পেজ রিকোয়েস্টে 404.html টেমপ্লেট রিটার্ন করুন
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# কাস্টম 405

webman-framework 1.5.23 থেকে, ফ্যালব্যাক কলব্যাক `status` প্যারামিটার সাপোর্ট করে। 404 মানে রিকোয়েস্ট নেই, 405 মানে বর্তমান রিকোয়েস্ট মেথড সাপোর্ট করা হয় না (উদাহরণ: `Route::post()` দিয়ে সেট করা রাউটে GET দিয়ে অ্যাক্সেস করা)।

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

# কাস্টম 500

**`app/view/500.html` তৈরি করুন**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
কাস্টম এরর টেমপ্লেট:
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**`app/exception/Handler.php` তৈরি করুন** (ডিরেক্টরি না থাকলে নিজে তৈরি করুন)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * রেন্ডারিং রিটার্ন
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // ajax রিকোয়েস্টে JSON ডেটা রিটার্ন করুন
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // পেজ রিকোয়েস্টে 500.html টেমপ্লেট রিটার্ন করুন
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**`config/exception.php` কনফিগার করুন**
```php
return [
    '' => \app\exception\Handler::class,
];
```
