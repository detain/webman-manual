# কন্ট্রোলার

PSR4 শৈলী অনুযায়ী, কন্ট্রোলার ক্লাসের নেমস্পেস শুরু হয় `plugin\{প্লাগইন আইডেন্টিফায়ার}` দিয়ে, উদাহরণস্বরূপ

নতুন কন্ট্রোলার ফাইল তৈরি করুন `plugin/foo/app/controller/FooController.php`।

```php
<?php
namespace plugin\foo\app\controller;

use support\Request;

class FooController
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

`http://127.0.0.1:8787/app/foo/foo`-এ অ্যাক্সেস করলে পেজটি `hello index` ফিরিয়ে দেবে

`http://127.0.0.1:8787/app/foo/foo/hello`-এ অ্যাক্সেস করলে পেজটি `hello webman` ফিরিয়ে দেবে


## URL অ্যাক্সেস
অ্যাপ্লিকেশন প্লাগইনের URL ঠিকানার পথ সবই `/app` দিয়ে শুরু হয়, তারপর প্লাগইন আইডেন্টিফায়ার এবং তারপর নির্দিষ্ট কন্ট্রোলার ও মেথড।
উদাহরণস্বরূপ `plugin\foo\app\controller\UserController`-এর URL ঠিকানা হল `http://127.0.0.1:8787/app/foo/user`
