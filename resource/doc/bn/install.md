# webman কীভাবে ইনস্টল করবেন

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux: PHP + Composer পরিবেশ ইনস্টল করুন (ইতিমধ্যে থাকলে এড়িয়ে যান)
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **দ্রষ্টব্য**
> উপরের কমান্ড Linux এবং macOS এর জন্য প্রযোজ্য। Windows ব্যবহারকারীদের আলাদাভাবে PHP ইনস্টল করতে হবে।

আপনি webman কর্তৃক প্রদত্ত [স্ট্যাটিক PHP](https://www.workerman.net/download) ম্যানুয়ালি ডাউনলোড করে এক্সট্রাক্ট করেও ব্যবহার করতে পারেন।

## 1. প্রকল্প তৈরি করুন

```php
composer create-project workerman/webman:~2.0
```

> **পরামর্শ**
> ত্রুটি হলে আপনি সমস্যাযুক্ত Composer মিরর ব্যবহার করছেন হতে পারেন। প্রক্সি অপসারণ করতে `composer config -g --unset repos.packagist` চালান।

## 2. চালান

webman ডিরেক্টরিতে যান

#### Windows ব্যবহারকারী
শুরু করতে `windows.bat` এ দ্বিগুণ ক্লিক করুন অথবা `php windows.php` চালান

> **পরামর্শ**
> ত্রুটি থাকলে ফাংশন নিষ্ক্রিয় করা থাকতে পারে। নিষ্ক্রিয়তা বাতিল করতে [ফাংশন নিষ্ক্রিয় পরীক্ষা](others/disable-function-check.md) দেখুন।

#### Linux ব্যবহারকারী
**ডিবাগ মোড** (উন্নয়নের জন্য: আউটপুট টার্মিনালে দেখা যায়; টার্মিনাল বন্ধ করলে webman সেবা বন্ধ হয়ে যায়)

```php
php start.php start
```

**ডেমন মোড** (প্রোডাকশনের জন্য: টার্মিনালে আউটপুট দেখা যায় না; টার্মিনাল বন্ধের পরও webman সেবা চলতে থাকে)

```php
php start.php start -d
```

#### Docker ব্যবহারকারী

সমস্ত সেবা চালু করুন এবং কনসোলে সংযুক্ত করুন
```php
docker-compose up
```

ব্যাকগ্রাউন্ড মোডে সেবা চালান
```php
docker-compose up -d
```

> **পরামর্শ**
> ত্রুটি থাকলে ফাংশন নিষ্ক্রিয় করা থাকতে পারে। নিষ্ক্রিয়তা বাতিল করতে [ফাংশন নিষ্ক্রিয় পরীক্ষা](others/disable-function-check.md) দেখুন।

## 3. অ্যাক্সেস

ব্রাউজারে `http://ip-ঠিকানা:8787` এ যান।
