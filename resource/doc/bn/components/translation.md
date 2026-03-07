# বহুভাষিক

বহুভাষিক সাপোর্ট [symfony/translation](https://github.com/symfony/translation) কম্পোনেন্ট ব্যবহার করে।

## ইনস্টলেশন
```
composer require symfony/translation
```

## ভাষা প্যাকেজ তৈরি করা
webman ডিফল্টভাবে ভাষা প্যাকেজ `resource/translations` ফোল্ডারে সংরক্ষণ করে (না থাকলে নিজে তৈরি করুন)। ফোল্ডার পরিবর্তন করতে `config/translation.php`-এ কনফিগার করুন।
প্রতিটি ভাষার একটি সাবফোল্ডার রয়েছে, ভাষা সংজ্ঞা ডিফল্টভাবে `messages.php`-এ রাখা হয়। উদাহরণ:
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

সমস্ত ভাষা ফাইল একটি অ্যারে রিটার্ন করে, উদাহরণ:
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## কনফিগারেশন

`config/translation.php`

```php
return [
    // ডিফল্ট ভাষা
    'locale' => 'zh_CN',
    // ফলব্যাক ভাষা: বর্তমান ভাষায় অনুবাদ না পাওয়া গেলে ফলব্যাক ভাষার অনুবাদ চেষ্টা করা হয়
    'fallback_locale' => ['zh_CN', 'en'],
    // ভাষা ফাইল সংরক্ষণের ফোল্ডার
    'path' => base_path() . '/resource/translations',
];
```

## অনুবাদ

অনুবাদের জন্য `trans()` মেথড ব্যবহার করুন।

ভাষা ফাইল `resource/translations/zh_CN/messages.php` তৈরি করুন:
```php
return [
    'hello' => '你好 世界!',
];
```

ফাইল `app/controller/UserController.php` তৈরি করুন:
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

`http://127.0.0.1:8787/user/get` এ এক্সেস করলে "你好 世界!" ফেরত আসবে।

## ডিফল্ট ভাষা পরিবর্তন

ভাষা পরিবর্তনের জন্য `locale()` মেথড ব্যবহার করুন।

ভাষা ফাইল `resource/translations/en/messages.php` যোগ করুন:
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
        // ভাষা পরিবর্তন
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
`http://127.0.0.1:8787/user/get` এ এক্সেস করলে "hello world!" ফেরত আসবে।

`trans()` ফাংশনের ৪র্থ প্যারামিটার দিয়ে সাময়িকভাবে ভাষা পরিবর্তনও করতে পারেন। উপরের উদাহরণ এবং নিচেরটি সমতুল্য:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // ৪র্থ প্যারামিটার দিয়ে ভাষা পরিবর্তন
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## প্রতিটি রিকোয়েস্টে স্পষ্টভাবে ভাষা সেট করা
translation একটি সিংগলটন, অর্থাৎ সব রিকোয়েস্ট একই ইন্সট্যান্স শেয়ার করে। কোনো রিকোয়েস্ট `locale()` দিয়ে ডিফল্ট ভাষা সেট করলে সেই প্রক্রিয়ার পরবর্তী সব রিকোয়েস্টে প্রভাব পড়ে। তাই প্রতিটি রিকোয়েস্টে স্পষ্টভাবে ভাষা সেট করতে হবে। যেমন নিচের মিডলওয়্যার ব্যবহার করুন:

ফাইল `app/middleware/Lang.php` তৈরি করুন (ফোল্ডার না থাকলে তৈরি করুন):
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

`config/middleware.php`-এ গ্লোবাল মিডলওয়্যার যোগ করুন:
```php
return [
    // গ্লোবাল মিডলওয়্যার
    '' => [
        // ... অন্যান্য মিডলওয়্যার বাদ
        app\middleware\Lang::class,
    ]
];
```


## প্লেসহোল্ডার ব্যবহার
কখনো বার্তায় অনুবাদযোগ্য ভেরিয়েবল থাকে, যেমন
```php
trans('hello ' . $name);
```
এক্ষেত্রে প্লেসহোল্ডার ব্যবহার করুন।

`resource/translations/zh_CN/messages.php` আপডেট করুন:
```php
return [
    'hello' => '你好 %name%!',
];
```
অনুবাদ করার সময় দ্বিতীয় প্যারামিটার দিয়ে প্লেসহোল্ডারের মান পাস করুন:
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## বহুবচন পরিচালনা
কিছু ভাষায় পরিমাণ অনুযায়ী বাক্য গঠন আলাদা। যেমন `There is %count% apple` তখনই সঠিক যখন `%count%` ১, ১-এর বেশি হলে ভুল।

এক্ষেত্রে **পাইপ** (`|`) দিয়ে বহুবচন রূপ লিখুন।

ভাষা ফাইল `resource/translations/en/messages.php`-এ `apple_count` যোগ করুন:
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

সংখ্যার রেঞ্জ দিয়েও জটিল বহুবচন নিয়ম লিখতে পারি:
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## ভাষা ফাইল নির্দিষ্ট করা

ডিফল্ট ভাষা ফাইলের নাম `messages.php`, তবে অন্য নামের ফাইলও তৈরি করতে পারেন।

ভাষা ফাইল `resource/translations/zh_CN/admin.php` তৈরি করুন:
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

`trans()`-এর তৃতীয় প্যারামিটার দিয়ে ভাষা ফাইল নির্দিষ্ট করুন (`.php` এক্সটেনশন বাদ দিন)।
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## আরও তথ্য
[symfony/translation ম্যানুয়াল](https://symfony.com/doc/current/translation.html) দেখুন
