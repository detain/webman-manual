# প্রসেস মনিটরিং
webman একটি অন্তর্নির্মিত মনিটর প্রসেস সহ আসে যা দুটি ফাংশন সমর্থন করে:
1. ফাইল আপডেট মনিটর করে এবং নতুন বিজনেস কোড স্বয়ংক্রিয়ভাবে রিলোড করে (সাধারণত উন্নয়নের সময় ব্যবহার করা হয়)
2. সমস্ত প্রসেসের মেমোরি ব্যবহার মনিটর করে; কোনও প্রসেস যদি `php.ini` এর `memory_limit` সীমা অতিক্রম করতে যাচ্ছে হয়, তাহলে সেই প্রসেস স্বয়ংক্রিয়ভাবে নিরাপদে পুনরায় চালু হয় (ব্যবসায় প্রভাব ছাড়াই)

## মনিটরিং কনফিগারেশন
`config/process.php` এ `monitor` কনফিগারেশন:
```php

global $argv;

return [
    // ফাইল আপডেট সনাক্তকরণ এবং স্বয়ংক্রিয় রিলোড
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // এই ডিরেক্টরিগুলো মনিটর করুন
            'monitorDir' => array_merge([    // কোন ডিরেক্টরির ফাইল মনিটর করতে হবে
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // এই এক্সটেনশনযুক্ত ফাইলগুলো মনিটর করা হবে
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // ফাইল মনিটরিং সক্ষম করা হবে কিনা
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // মেমোরি মনিটরিং সক্ষম করা হবে কিনা
            ]
        ]
    ]
];
```
`monitorDir` আপডেটের জন্য কোন ডিরেক্টরি মনিটর করা হবে তা কনফিগার করে (মনিটর করা ডিরেক্টরিতে ফাইলের সংখ্যা অত্যধিক হওয়া উচিত নয়)।
`monitorExtensions` `monitorDir` ডিরেক্টরিতে কোন ফাইল এক্সটেনশন মনিটর করা হবে তা কনফিগার করে।
`options.enable_file_monitor` যখন `true` হয়, ফাইল আপডেট মনিটরিং সক্ষম হয় (লিনাক্সে ডিবাগ মোডে চালানোর সময় ডিফল্টরূপে ফাইল মনিটরিং সক্ষম থাকে)।
`options.enable_memory_monitor` যখন `true` হয়, মেমোরি মনিটরিং সক্ষম হয় (উইন্ডোজে মেমোরি মনিটরিং সমর্থিত নয়)।

> **টিপ**
> উইন্ডোজে, ফাইল আপডেট মনিটরিং শুধুমাত্র `windows.bat` বা `php windows.php` চালানোর সময় সক্ষম হয়।
