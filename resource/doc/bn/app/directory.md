# ডিরেক্টরি কাঠামো

```
plugin/
└── foo
    ├── app
    │   ├── controller
    │   │   └── IndexController.php
    │   ├── exception
    │   │   └── Handler.php
    │   ├── functions.php
    │   ├── middleware
    │   ├── model
    │   └── view
    │       └── index
    │           └── index.html
    ├── config
    │   ├── app.php
    │   ├── autoload.php
    │   ├── container.php
    │   ├── database.php
    │   ├── exception.php
    │   ├── log.php
    │   ├── middleware.php
    │   ├── process.php
    │   ├── redis.php
    │   ├── route.php
    │   ├── static.php
    │   ├── thinkorm.php
    │   ├── translation.php
    │   └── view.php
    ├── public
    └── api
```

একটি অ্যাপ্লিকেশন প্লাগইনের webman-এর মতো একই ডিরেক্টরি কাঠামো এবং কনফিগারেশন ফাইল রয়েছে। বাস্তবে, সাধারণ webman অ্যাপ্লিকেশন বিকাশের সাথে উন্নয়ন অভিজ্ঞতা প্রায় অভিন্ন।

প্লাগইন ডিরেক্টরি এবং নামকরণ PSR-4 স্পেসিফিকেশন অনুসরণ করে। প্লাগইনগুলি সব `plugin` ডিরেক্টরিতে থাকে বলে, নামস্থানগুলো `plugin` দিয়ে শুরু হয়, উদাহরণস্বরূপ `plugin\foo\app\controller\UserController`।

## api ডিরেক্টরি সম্পর্কে

প্রতিটি প্লাগইনে একটি `api` ডিরেক্টরি রয়েছে। যদি আপনার অ্যাপ্লিকেশন অন্যান্য অ্যাপ্লিকেশন দ্বারা কল করার জন্য অভ্যন্তরীণ ইন্টারফেস প্রদান করে, তবে সেই ইন্টারফেসগুলো `api` ডিরেক্টরিতে রাখুন।

দ্রষ্টব্য: এখানে উল্লিখিত ইন্টারফেসগুলি ফাংশন কল ইন্টারফেস, নেটওয়ার্ক/HTTP ইন্টারফেস নয়।

উদাহরণস্বরূপ, ইমেইল প্লাগইন `plugin/email/api/Email.php`-এ `Email::send()` ইন্টারফেস সরবরাহ করে যাতে অন্যান্য অ্যাপ্লিকেশন ইমেইল পাঠানোর সময় কল করতে পারে। এছাড়াও, `plugin/email/api/Install.php` webman-admin প্লাগইন মার্কেট দ্বারা ইনস্টল বা আনইনস্টল অপারেশন চালানোর জন্য স্বয়ংক্রিয়ভাবে তৈরি হয়।
