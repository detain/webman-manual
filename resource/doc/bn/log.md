# লগ
webman [monolog/monolog](https://github.com/Seldaek/monolog) ব্যবহার করে লগ প্রসেস করে।

## ব্যবহার
```php
<?php
namespace app\controller;

use support\Request;
use support\Log;

class FooController
{
    public function index(Request $request)
    {
        Log::info('log test');
        return response('hello index');
    }
}
```

## প্রদানকৃত মেথড
```php
Log::log($level, $message, array $context = [])
Log::debug($message, array $context = [])
Log::info($message, array $context = [])
Log::notice($message, array $context = [])
Log::warning($message, array $context = [])
Log::error($message, array $context = [])
Log::critical($message, array $context = [])
Log::alert($message, array $context = [])
Log::emergency($message, array $context = [])
```
এর সমতুল্য
```php
$log = Log::channel('default');
$log->log($level, $message, array $context = [])
$log->debug($message, array $context = [])
$log->info($message, array $context = [])
$log->notice($message, array $context = [])
$log->warning($message, array $context = [])
$log->error($message, array $context = [])
$log->critical($message, array $context = [])
$log->alert($message, array $context = [])
$log->emergency($message, array $context = [])
```

## কনফিগারেশন
```php
return [
    // ডিফল্ট লগ চ্যানেল
    'default' => [
        // ডিফল্ট চ্যানেলে হ্যান্ডলার প্রসেস করার, এটা অনেক সেট করা যাবে
        'handlers' => [
            [   
                // হ্যান্ডলার ক্লাসের নাম
                'class' => Monolog\Handler\RotatingFileHandler::class,
                // হ্যান্ডলার ক্লাসের কন্সট্রাক্টর প্যারামিটার
                'constructor' => [
                    runtime_path() . '/logs/webman.log',
                    Monolog\Logger::DEBUG,
				],
                // ফর্ম্যাটার সম্পর্কে
                'formatter' => [
                    // ফর্ম্যাটার প্রসেসিং ক্লাসের নাম
                    'class' => Monolog\Formatter\LineFormatter::class,
                    // ফর্ম্যাটার প্রসেসিং ক্লাসের কন্সট্রাক্টর প্যারামিটার
                    'constructor' => [ null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
];
```

## মাল্টি চ্যানেল
monolog একাধিক চ্যানেল সমর্থন করে, ডিফল্ট হিসেবে `default` চ্যানেল ব্যবহার করা হয়। আপনি `log2` চ্যানেল যুক্ত করতে চাইলে, কনফিগারেশন নিম্নরূপ হবে:
```php
return [
    // ডিফল্ট লগ চ্যানেল
    'default' => [
        // ডিফল্ট চ্যানেলে হ্যান্ডলার প্রসেস করার, এটা অনেক সেট করা যাবে
        'handlers' => [
            [   
                // হ্যান্ডলার ক্লাসের নাম
                'class' => Monolog\Handler\RotatingFileHandler::class,
                // হ্যান্ডলার ক্লাসের কন্সট্রাক্টর প্যারামিটার
                'constructor' => [
                    runtime_path() . '/logs/webman.log',
                    Monolog\Logger::DEBUG,
				],
                // ফর্ম্যাটার সম্পর্কে
                'formatter' => [
                    // ফর্ম্যাটার প্রসেসিং ক্লাসের নাম
                    'class' => Monolog\Formatter\LineFormatter::class,
                    // ফর্ম্যাটার প্রসেসিং ক্লাসের কন্সট্রাক্টর প্যারামিটার
                    'constructor' => [ null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
    // log2 চ্যানেল
    'log2' => [
        // log2 চ্যানেলের হ্যান্ডলার প্রসেস করার জন্য, একাধিক সেট করা যাবে
        'handlers' => [
            [   
                // হ্যান্ডলার ক্লাসের নাম
                'class' => Monolog\Handler\RotatingFileHandler::class,
                // হ্যান্ডলার ক্লাসের কন্সট্রাক্টর প্যারামিটার
                'constructor' => [
                    runtime_path() . '/logs/log2.log',
                    Monolog\Logger::DEBUG,
				],
                // ফর্ম্যাটার সম্পর্কে
                'formatter' => [
                    // ফর্ম্যাটার প্রসেসিং ক্লাসের নাম
                    'class' => Monolog\Formatter\LineFormatter::class,
                    // ফর্ম্যাটার প্রসেসিং ক্লাসের কন্সট্রাক্টর প্যারামিটার
                    'constructor' => [ null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
];
```

`log2` চ্যানেল ব্যবহার করার পদ্ধতি নিম্নরূপ:
```php
<?php
namespace app\controller;

use support\Request;
use support\Log;

class FooController
{
    public function index(Request $request)
    {
        $log = Log::channel('log2');
        $log->info('log2 test');
        return response('hello index');
    }
}
```
