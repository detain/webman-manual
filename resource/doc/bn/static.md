# webman স্থির ফাইল প্রসেসিং

webman স্থির ফাইল অ্যাক্সেস সাপোর্ট করে, স্থির ফাইলগুলি `public` ডিরেক্টরিতে রাখা হয়। উদাহরণস্বরূপ, `http://127.0.0.8787/upload/avatar.png` অ্যাক্সেস করা প্রকৃতপক্ষে `{প্রধান প্রজেক্ট ডিরেক্টরি}/public/upload/avatar.png` অ্যাক্সেস করা।

> **লক্ষ্য করুন**
> `/app/xx/ফাইলের_নাম` দিয়ে শুরু হওয়া স্থির ফাইল অ্যাক্সেস প্রকৃতপক্ষে অ্যাপ্লিকেশন প্লাগইনের `public` ডিরেক্টরিতে অ্যাক্সেস করে। অর্থাৎ `{প্রধান প্রজেক্ট ডিরেক্টরি}/public/app/` এর অধীনে ডিরেক্টরি অ্যাক্সেস সাপোর্ট করা হয় না।
> আরও জানতে [অ্যাপ্লিকেশন প্লাগইন](./plugin/app.md) দেখুন।

## স্থির ফাইল সাপোর্ট বন্ধ করুন

যদি স্থির ফাইল সাপোর্টের প্রয়োজন না থাকে, `config/static.php` খুলে `enable` অপশন false-এ পরিবর্তন করুন। বন্ধ করার পর সমস্ত স্থির ফাইল অ্যাক্সেস 404 রিটার্ন করবে।

## স্থির ফাইল ডিরেক্টরি পরিবর্তন করুন

webman ডিফল্টভাবে স্থির ফাইলের জন্য `public` ডিরেক্টরি ব্যবহার করে। পরিবর্তন করতে `support/helpers.php` ফাইলের `public_path()` হেল্পার ফাংশন সম্পাদনা করুন।

## স্থির ফাইল মিডলওয়্যার

webman `app/middleware/StaticFile.php`-এ অবস্থিত একটি স্থির ফাইল মিডলওয়্যার নিয়ে আসে।
কখনও কখনও স্থির ফাইলগুলিতে কিছু প্রসেসিং করতে হয়, যেমন স্থির ফাইলে ক্রস-অরিজিন HTTP হেডার যোগ করা বা ডট (`.`) দিয়ে শুরু হওয়া ফাইলে অ্যাক্সেস নিষিদ্ধ করা—এই মিডলওয়্যার ব্যবহার করা যেতে পারে।

`app/middleware/StaticFile.php` এর বিষয়বস্তু নিম্নরূপ:
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
        // বিন্দু দিয়ে শুরু হওয়া লুকানো ফাইলে অ্যাক্সেস নিষিদ্ধ
        if (strpos($request->path(), '/.') !== false) {
            return response('<h1>403 forbidden</h1>', 403);
        }
        /** @var Response $response */
        $response = $next($request);
        // ক্রস-অরিজিন হেডার যোগ করুন
        /*$response->withHeaders([
            'Access-Control-Allow-Origin'      => '*',
            'Access-Control-Allow-Credentials' => 'true',
        ]);*/
        return $response;
    }
}
```
এই মিডলওয়্যার প্রয়োজন হলে, `config/static.php` এর `middleware` অপশনে সক্রিয় করুন।
