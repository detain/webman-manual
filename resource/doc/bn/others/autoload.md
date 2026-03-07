# স্বয়ংক্রিয় লোডিং

## Composer দিয়ে PSR-0 সমর্থিত ফাইল লোড করা
webman `PSR-4` স্বয়ংক্রিয় লোডিং স্পেসিফিকেশন অনুসরণ করে। আপনার প্রজেক্টে PSR-0 সমর্থিত লাইব্রেরি লোড করা প্রয়োজন হলে এই ধাপগুলো অনুসরণ করুন:

- PSR-0 লাইব্রেরি রাখতে `extend` ডিরেক্টরি তৈরি করুন
- `composer.json` সম্পাদনা করুন এবং `autoload` এর নিচে এটি যোগ করুন:

```json
"psr-0" : {
    "": "extend/"
}
```
চূড়ান্ত ফলাফল এরকম হবে:
![](../../assets/img/psr0.png)

- `composer dumpautoload` চালান
- webman পুনরায় চালু করতে `php start.php restart` চালান (দ্রষ্টব্য: পরিবর্তন কার্যকর হতে পুনরায় চালু করা আবশ্যক)

## Composer দিয়ে কিছু ফাইল লোড করা

- `composer.json` সম্পাদনা করুন এবং `autoload.files` এ লোড করার ফাইলগুলো যোগ করুন:
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- `composer dumpautoload` চালান
- webman পুনরায় চালু করতে `php start.php restart` চালান (দ্রষ্টব্য: পরিবর্তন কার্যকর হতে পুনরায় চালু করা আবশ্যক)

> **দ্রষ্টব্য**
> composer.json এ `autoload.files` এর কনফিগার করা ফাইলগুলো webman চালু হওয়ার আগে লোড হয়। অপরদিকে ফ্রেমওয়ার্কের `config/autoload.php` দিয়ে লোড করা ফাইলগুলো webman চালুর পরে লোড হয়।
> composer.json এর `autoload.files` ফাইলগুলো পরিবর্তনের পর restart করলে কাজে আসবে; reload কাজ করবে না। `config/autoload.php` দিয়ে লোড করা ফাইলগুলো hot-reload সাপোর্ট করে; reload করলে পরিবর্তন কার্যকর হয়।

## ফ্রেমওয়ার্ক দিয়ে কিছু ফাইল লোড করা
কিছু ফাইল PSR স্পেসিফিকেশন মেনে চলে না এবং স্বয়ংক্রিয় লোড হয় না। `config/autoload.php` কনফিগার করে এগুলো লোড করতে পারেন, উদাহরণ:
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **দ্রষ্টব্য**
 > `autoload.php` এ `support/Request.php` এবং `support/Response.php` লোড করার সেটিং আছে কারণ `vendor/workerman/webman-framework/src/support/` এও একই নামের ফাইল আছে। `autoload.php` দিয়ে প্রজেক্ট রুটের ফাইলগুলো অগ্রাধিকার পায়, ফলে `vendor` ফাইল না বদলিয়েও এই দুটি কাস্টমাইজ করা যায়। কাস্টমাইজের প্রয়োজন না থাকলে এই দুটি কনফিগারেশন বাদ দিতে পারেন।
