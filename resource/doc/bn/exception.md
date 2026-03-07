# ব্যতিক্রম ব্যবস্থাপনা

## কনফিগারেশন
`config/exception.php`
```php
return [
    // এখানে ব্যতিক্রম ব্যবস্থাপনা ক্লাস কনফিগার করুন
    '' => support\exception\Handler::class,
];
```
মাল্টি-অ্যাপ মোডে প্রতিটি অ্যাপ্লিকেশনের জন্য আলাদাভাবে ব্যতিক্রম ব্যবস্থাপনা ক্লাস কনফিগার করতে পারবেন, [মাল্টি-অ্যাপ](multiapp.md) দেখুন।


## ডিফল্ট ব্যতিক্রম ব্যবস্থাপনা ক্লাস
webman এ ব্যতিক্রম ডিফল্টরূপে `support\exception\Handler` ক্লাস দ্বারা ব্যবস্থাপনা করা হয়। কনফিগারেশন ফাইল `config/exception.php` সম্পাদনা করে ডিফল্ট ব্যতিক্রম ব্যবস্থাপনা ক্লাস পরিবর্তন করতে পারেন। ব্যতিক্রম ব্যবস্থাপনা ক্লাসকে `Webman\Exception\ExceptionHandlerInterface` ইন্টারফেস ইমপ্লিমেন্ট করতে হবে।
```php
interface ExceptionHandlerInterface
{
    /**
     * লগ রেকর্ড করুন
     * @param Throwable $e
     * @return mixed
     */
    public function report(Throwable $e);

    /**
     * প্রতিক্রিয়া রেন্ডার করুন
     * @param Request $request
     * @param Throwable $e
     * @return Response
     */
    public function render(Request $request, Throwable $e) : Response;
}
```



## প্রতিক্রিয়া রেন্ডারিং
ব্যতিক্রম ব্যবস্থাপনা ক্লাসের `render` মেথড প্রতিক্রিয়া রেন্ডার করার জন্য ব্যবহৃত হয়।

কনফিগারেশন ফাইল `config/app.php` এ `debug` এর মান যদি `true` হয় (পরবর্তীতে `app.debug=true` হিসাবে উল্লেখ করা হবে), বিস্তারিত ব্যতিক্রম তথ্য রিটার্ন হবে, অন্যথায় সংক্ষিপ্ত ব্যতিক্রম তথ্য রিটার্ন হবে।

যদি অনুরোধ json প্রত্যাশা করে, তবে ব্যতিক্রম তথ্য json ফরম্যাটে রিটার্ন হবে, যেমন
```json
{
    "code": "500",
    "msg": "ব্যতিক্রম তথ্য"
}
```
যদি `app.debug=true` হয়, json ডেটায় অতিরিক্তভাবে `trace` ফিল্ড যোগ হবে যা বিস্তারিত কল স্ট্যাক রিটার্ন করে।

আপনি নিজস্ব ব্যতিক্রম ব্যবস্থাপনা ক্লাস লিখে ডিফল্ট ব্যতিক্রম ব্যবস্থাপনা লজিক পরিবর্তন করতে পারেন।

# ব্যবসায়িক ব্যতিক্রম BusinessException
কখনো কখনো আমরা কোনো নেস্টেড ফাংশনে অনুরোধ বাতিল করে ক্লায়েন্টকে একটি ত্রুটি বার্তা ফেরত দিতে চাই, এসময় `BusinessException` ছুঁড়ে এটা করা যায়।
উদাহরণ:

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
            throw new BusinessException('প্যারামিটার ত্রুটি', 3000);
        }
    }
}
```

উপরের উদাহরণ নিচের মত রিটার্ন করবে
```json
{"code": 3000, "msg": "প্যারামিটার ত্রুটি"}
```

> **দ্রষ্টব্য**
> ব্যবসায়িক ব্যতিক্রম BusinessException কে try দিয়ে ক্যাচ করার প্রয়োজন নেই, ফ্রেমওয়ার্ক স্বয়ংক্রিয়ভাবে ক্যাচ করবে এবং অনুরোধের ধরন অনুযায়ী উপযুক্ত আউটপুট রিটার্ন করবে।

## কাস্টম ব্যবসায়িক ব্যতিক্রম

যদি উপরের প্রতিক্রিয়া আপনার চাহিদা পূরণ না করে, যেমন `msg` কে `message` এ পরিবর্তন করতে চান, তাহলে আপনি একটি কাস্টম `MyBusinessException` তৈরি করতে পারেন।

নতুন ফাইল `app/exception/MyBusinessException.php` তৈরী করুন নিচের বিষয়বস্তু দিয়ে
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
        // json অনুরোধে json ডেটা রিটার্ন করুন
        if ($request->expectsJson()) {
            return json(['code' => $this->getCode() ?: 500, 'message' => $this->getMessage()]);
        }
        // অ-json অনুরোধে একটি পৃষ্ঠা রিটার্ন করুন
        return new Response(200, [], $this->getMessage());
    }
}
```

এভাবে যখন ব্যবসায়িক লজিক কলে
```php
use app\exception\MyBusinessException;

throw new MyBusinessException('প্যারামিটার ত্রুটি', 3000);
```
json অনুরোধ নিচের মত json রিটার্ন পাবে
```json
{"code": 3000, "message": "প্যারামিটার ত্রুটি"}
```

> **পরামর্শ**
> যেহেতু BusinessException ব্যবসায়িক ব্যতিক্রমের অন্তর্ভুক্ত (যেমন ব্যবহারকারী ইনপুট প্যারামিটার ত্রুটি), এটি পূর্বনির্ধারিত, তাই ফ্রেমওয়ার্ক একে 치명িক ত্রুটি বিবেচনা করবে না এবং লগে রেকর্ড করবে না।

## সারাংশ
যেকোনো সময় যখন বর্তমান অনুরোধ বাতিল করে ক্লায়েন্টকে তথ্য ফেরত দিতে চান, তখন `BusinessException` ব্যবহার বিবেচনা করতে পারেন।
