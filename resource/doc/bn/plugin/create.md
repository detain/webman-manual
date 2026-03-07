# বেসিক প্লাগইন তৈরির ও প্রকাশের প্রক্রিয়া

## মূলনীতি
1. ক্রস-অরিজিন প্লাগইন ধরা যাক, প্লাগইন তিন ভাগে বিভক্ত: ক্রস-অরিজিন মিডলওয়্যার ফাইল, কনফিগ ফাইল `middleware.php`, এবং কমান্ড দিয়ে স্বয়ংক্রিয়ভাবে তৈরি `Install.php`।
2. এই তিন ফাইল প্যাকেজ করে Composer-এ প্রকাশ করতে কমান্ড ব্যবহার করা হয়।
3. ব্যবহারকারী Composer দিয়ে ক্রস-অরিজিন প্লাগইন ইনস্টল করলে, `Install.php` মিডলওয়্যার ও কনফিগ কপি করে `{মূল প্রজেক্ট}/config/plugin`-এ রাখে যাতে webman লোড করতে পারে এবং ক্রস-অরিজিন সক্রিয় হয়।
4. ব্যবহারকারী Composer দিয়ে প্লাগইন সরিয়ে দিলে, `Install.php` সংশ্লিষ্ট মিডলওয়্যার ও কনফিগ ফাইল মুছে দেয় এবং প্লাগইন স্বয়ংক্রিয়ভাবে আনইনস্টল হয়।

## বিধিনিষেধ
1. প্লাগইনের নাম দুই অংশে: `vendor` এবং `প্লাগইনের নাম`, যেমন `webman/push`, Composer প্যাকেজের নামের সাথে মিল থাকতে হবে।
2. প্লাগইন কনফিগ ফাইল `config/plugin/vendor/প্লাগইনের নাম/`-এ রাখা হয় (কনসোল কমান্ড কনফিগ ডিরেক্টরি নিজে তৈরি করে)। প্লাগইনের কনফিগ লাগে না হলে, স্বয়ংক্রিয় তৈরিকৃত ডিরেক্টরি মুছে ফেলতে হবে।
3. প্লাগইন কনফিগ ডিরেক্টরি শুধু সমর্থন করে: `app.php` (প্রধান কনফিগ), `bootstrap.php` (প্রক্রিয়া চালু), `route.php` (রুট), `middleware.php` (মিডলওয়্যার), `process.php` (কাস্টম প্রক্রিয়া), `database.php` (ডাটাবেস), `redis.php` (Redis), `thinkorm.php` (thinkorm)। এগুলো webman নিজে চিনে নেয়।
4. কনফিগ অ্যাক্সেস: `config('plugin.vendor.প্লাগইনের নাম.কনফিগ ফাইল.আইটেম');`, যেমন `config('plugin.webman.push.app.app_key')`।
5. প্লাগইনের নিজস্ব ডাটাবেস কনফিগ থাকলে: `illuminate/database`-এর জন্য `Db::connection('plugin.vendor.প্লাগইনের নাম.সংযোগ')`, `thinkorm`-এর জন্য `Db::connect('plugin.vendor.প্লাগইনের নাম.সংযোগ')` ব্যবহার করুন।
6. প্লাগইন `app/`-এ বিজনেস ফাইল রাখতে চাইলে, মূল প্রজেক্ট ও অন্যান্য প্লাগইনের সঙ্গে সংঘর্ষ এড়াতে হবে।
7. প্লাগইন মূল প্রজেক্টে ফাইল বা ফোল্ডার কপি না করাই ভালো। যেমন ক্রস-অরিজিন প্লাগইনে কেবল কনফিগ কপি হয়; মিডলওয়্যার ফাইল `vendor/webman/cros/src`-এ থাকে।
8. প্লাগইন নেমস্পেস PascalCase ব্যবহারের পরামর্শ, যেমন `Webman/Console`।

## উদাহরণ

**`webman/console` কমান্ডলাইন ইনস্টল করুন**

`composer require webman/console`

### প্লাগইন তৈরি করুন

ধরি তৈরি করতে চাওয়া প্লাগইনের নাম `foo/admin` (Composer-এ প্রকাশের প্রজেক্ট নামও এটাই, ছোট হাতের হতে হবে)। চালান:

`php webman plugin:create --name=foo/admin`

এতে `vendor/foo/admin` (প্লাগইন ফাইল) এবং `config/plugin/foo/admin` (কনফিগ) তৈরি হবে।

> নোট
> `config/plugin/foo/admin` সমর্থন করে: `app.php`, `bootstrap.php`, `route.php`, `middleware.php`, `process.php`, `database.php`, `redis.php`, `thinkorm.php`। webman-এর মতো ফরম্যাট, অটো মার্জ।
> অ্যাক্সেসে `plugin` প্রিফিক্স ব্যবহার করুন, যেমন `config('plugin.foo.admin.app')`।


### প্লাগইন এক্সপোর্ট করুন

ডেভেলপমেন্ট শেষে চালান:

`php webman plugin:export --name=foo/admin`

এক্সপোর্ট

> ব্যাখ্যা
> এক্সপোর্টে `config/plugin/foo/admin` কপি হয় `vendor/foo/admin/src`-এ এবং `Install.php` তৈরি হয়। ইনস্টল/আনইনস্টল সময় `Install.php` চলে।
> ডিফল্ট ইনস্টল: `vendor/foo/admin/src`-এর কনফিগ প্রজেক্টের `config/plugin`-এ কপি করে।
> ডিফল্ট আনইনস্টল: প্রজেক্টের `config/plugin` থেকে সংশ্লিষ্ট কনফিগ ফাইল মুছে দেয়।
> ইনস্টল/আনইনস্টলে কাস্টম লজিকের জন্য `Install.php` এডিট করা যায়।

### প্লাগইন জমা দিন
* ধরে নিন আপনার [GitHub](https://github.com) ও [Packagist](https://packagist.org) অ্যাকাউন্ট আছে।
* [GitHub](https://github.com)-এ `admin` রিপোজিটরি বানান ও কোড পুশ করুন, যেমন `https://github.com/আপনার-ইউজারনেম/admin`।
* `https://github.com/আপনার-ইউজারনেম/admin/releases/new`-এ গিয়ে release করুন, যেমন `v1.0.0`।
* [Packagist](https://packagist.org)-এ `Submit` ক্লিক করে `https://github.com/আপনার-ইউজারনেম/admin` পাঠান, প্লাগইন প্রকাশ হবে।

> **পরামর্শ**
> Packagist-এ নাম কনফ্লিক্ট হলে অন্য vendor নিন, যেমন `foo/admin` কে `myfoo/admin` করুন।

আপডেটে: কোড GitHub-এ পুশ করুন, `https://github.com/আপনার-ইউজারনেম/admin/releases/new`-এ নতুন release তৈরি করুন, তারপর `https://packagist.org/packages/foo/admin`-এ `Update` ক্লিক করুন।

## প্লাগইনে কমান্ড যোগ করুন
কিছু প্লাগইনে কাস্টম কমান্ড লাগে। যেমন `webman/redis-queue` ইনস্টল করলে প্রজেক্টে `redis-queue:consumer` কমান্ড পাওয়া যায়। `php webman redis-queue:consumer send-mail` চালালে দ্রুত `SendMail.php` consumer ক্লাস তৈরি হয়, ডেভেলপমেন্ট সহজ হয়।

`foo/admin` প্লাগইনে `foo-admin:add` কমান্ড যোগ করতে:

### কমান্ড তৈরি করুন

**`vendor/foo/admin/src/FooAdminAddCommand.php` ফাইল তৈরি করুন**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = 'কমান্ডের বিবরণ';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **নোট**
> প্লাগইনগুলোর কমান্ড কনফ্লিক্ট এড়াতে `vendor-plugin:কমান্ড` ফরম্যাট ব্যবহার করুন। যেমন `foo/admin`-এর সব কমান্ড `foo-admin:` প্রিফিক্স দিতে হবে, যেমন `foo-admin:add`।

### কনফিগ যোগ করুন
**`config/plugin/foo/admin/command.php` তৈরি করুন**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // প্রয়োজন অনুযায়ী আরও যোগ করুন...
];
```

> **পরামর্শ**
> `command.php` প্লাগইনের কাস্টম কমান্ড রেজিস্ট্রি করে। প্রতিটি এন্ট্রি একটি কমান্ড ক্লাস। `webman/console` স্বয়ংক্রিয়ভাবে লোড করে। বিস্তারিত [কনসোল কমান্ড](console.md) দেখুন।

### এক্সপোর্ট চালান
`php webman plugin:export --name=foo/admin` চালিয়ে প্লাগইন এক্সপোর্ট করে Packagist-এ প্রকাশ করুন। `foo/admin` ইনস্টল হলে `foo-admin:add` কমান্ড পাওয়া যাবে। `php webman foo-admin:add jerry` চালালে `Admin add jerry` প্রিন্ট হবে।
