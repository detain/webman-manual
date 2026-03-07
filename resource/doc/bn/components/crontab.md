# crontab সময়সূচি উপাদান

## বর্ণনা

`workerman/crontab` লিনাক্সের crontab-এর মতো, তবে সেকেন্ড স্তরে সময়সূচি সমর্থন করে।

সময় ফরম্যাট:

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[ঐচ্ছিক; না থাকলে ন্যূনতম একক মিনিট]
```

## প্রকল্প URL

https://github.com/walkor/crontab
  
## ইনস্টলেশন
 
```php
composer require workerman/crontab
```
  
## ব্যবহার

**ধাপ ১: প্রক্রিয়া ফাইল `app/process/Task.php` তৈরি করুন**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {

        // প্রতি সেকেন্ড চালান
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // প্রতি ৫ সেকেন্ডে চালান
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // প্রতি মিনিটে চালান
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // প্রতি ৫ মিনিটে চালান
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // প্রতি মিনিটের প্রথম সেকেন্ডে চালান
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // প্রতিদিন সকাল ৭টা ৫০ মিনিটে চালান (এখানে সেকেন্ড বাদ দেওয়া হয়েছে)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**ধাপ ২: webman-এর সাথে প্রক্রিয়া চালু করার কনফিগারেশন**
  
`config/process.php` ফাইল খুলে নিচের অংশ যোগ করুন:

```php
return [
    ....অন্যান্য কনফিগারেশন বাদ....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**ধাপ ৩: webman পুনরারম্ভ করুন**

> নোট: সময়সূচি কাজগুলো সঙ্গে সঙ্গে চলে না; পরবর্তী মিনিট থেকে শুরু হয়।

## নোট
crontab অ্যাসিঙ্ক্রোনাস নয়। যেমন: একটি task প্রক্রিয়ায় A ও B দুটি টাইমার আছে, দুটোই প্রতি সেকেন্ড চলে। কাজ A যদি ১০ সেকেন্ড নেয় তাহলে B কে A শেষ হওয়া পর্যন্ত অপেক্ষা করতে হয়, ফলে B-এর চালাতে বিলম্ব হয়।
সময় ব্যবধানে সংবেদনশীল হলে সংবেদনশীল সময়সূচি কাজ আলাদা প্রক্রিয়ায় চালান। উদাহরণ `config/process.php`:

```php
return [
    ....অন্যান্য কনফিগারেশন বাদ....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
সময়-সংবেদনশীল কাজ রাখুন `process/Task1.php`-এ, অন্যান্য রাখুন `process/Task2.php`-এ।

`config/process.php` সম্পর্কে আরও জানতে [কাস্টম প্রক্রিয়া](../process.md) দেখুন।
