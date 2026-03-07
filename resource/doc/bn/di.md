# ডিপেন্ডেন্সি অটো ইনজেকশন

webman-এ ডিপেন্ডেন্সি অটো ইনজেকশন একটি ঐচ্ছিক ফিচার এবং ডিফল্টভাবে বন্ধ থাকে। আপনার যদি ডিপেন্ডেন্সি অটো ইনজেকশন দরকার হয় তাহলে [php-di](https://php-di.org/doc/getting-started.html) ব্যবহারের সুপারিশ করা হয়। নিচে webman-এর সাথে `php-di` ব্যবহারের বিবরণ দেওয়া হল।

## ইনস্টলেশন

```
composer require php-di/php-di:^7.0
```

`config/container.php` কনফিগারেশন সম্পাদনা করুন। চূড়ান্ত কন্টেন্ট এরকম হওয়া উচিত:

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> `config/container.php` ফাইল শেষ পর্যন্ত PSR-11 স্পেসিফিকেশনের সাথে সামঞ্জস্যপূর্ণ একটি কনটেইনার ইনস্ট্যান্স রিটার্ন করবে। আপনি যদি `php-di` ব্যবহার না করতে চান তাহলে এখানে অন্য কোনো PSR-11 সামঞ্জস্যপূর্ণ কনটেইনার ইনস্ট্যান্স তৈরি ও রিটার্ন করতে পারেন। ডিফল্ট কনফিগারেশন শুধু webman-এর মৌলিক কনটেইনার ফাংশনালিটি দেয়।

## কনস্ট্রাক্টর ইনজেকশন

`app/service/Mailer.php` ফাইল তৈরি করুন (ফোল্ডার না থাকলে তৈরি করুন) এবং নিচের কন্টেন্ট দিন:

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // ইমেইল পাঠানোর কোড বাদ দেওয়া হয়েছে
    }
}
```

`app/controller/UserController.php` এর কন্টেন্ট:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{

    public function __construct(private Mailer $mailer)
    {
    }

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

সাধারণত `app\controller\UserController` ইনস্ট্যান্স করতে নিচের কোড লাগে:

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

`php-di` ব্যবহার করলে ডেভেলপারের কন্ট্রোলারে `Mailer` ম্যানুয়ালি ইনস্ট্যান্স করার দরকার নেই — webman নিজেই করবে। `Mailer` ইনস্ট্যান্স করার সময় যদি অন্য ক্লাস ডিপেন্ডেন্সি থাকে তাহলে webman সেগুলোও ইনস্ট্যান্স ও ইনজেক্ট করবে। ডেভেলপারের কোনো ইনিশিয়ালাইজেশন লাগে না।

> **মনে রাখুন**
> ডিপেন্ডেন্সি অটো ইনজেকশন শুধু ফ্রেমওয়ার্ক বা `php-di` দ্বারা তৈরি ইনস্ট্যান্সেই কাজ করে। `new` দিয়ে ম্যানুয়ালি তৈরি ইনস্ট্যান্সে কাজ করে না। ইনজেকশন চাইলে `new` এর বদলে `support\Container` ইন্টারফেস ব্যবহার করুন, যেমন:

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// new দিয়ে তৈরি ইনস্ট্যান্সে ডিপেন্ডেন্সি ইনজেকশন কাজ করে না
$user_service = new UserService;
// new দিয়ে তৈরি ইনস্ট্যান্সে ডিপেন্ডেন্সি ইনজেকশন কাজ করে না
$log_service = new LogService($path, $name);

// Container দিয়ে তৈরি ইনস্ট্যান্সে ডিপেন্ডেন্সি ইনজেকশন কাজ করে
$user_service = Container::get(UserService::class);
// Container দিয়ে তৈরি ইনস্ট্যান্সে ডিপেন্ডেন্সি ইনজেকশন কাজ করে
$log_service = Container::make(LogService::class, [$path, $name]);
```

## অ্যাট্রিবিউট ইনজেকশন

কনস্ট্রাক্টর ডিপেন্ডেন্সি ইনজেকশনের পাশাপাশি অ্যাট্রিবিউট ইনজেকশন ব্যবহার করা যায়। আগের উদাহরণের ধারাবাহিকতায় `app\controller\UserController` এভাবে পরিবর্তন করুন:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private Mailer $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

এ উদাহরণ ইনজেকশনের জন্য `#[Inject]` অ্যাট্রিবিউট ব্যবহার করে এবং অবজেক্ট টাইপ অনুযায়ী ইনস্ট্যান্স মেম্বার ভেরিয়েবলে অটো ইনজেক্ট করে। কনস্ট্রাক্টর ইনজেকশনের মতই ফলাফল, কিন্তু কোড আরও সংক্ষিপ্ত।

> **মনে রাখুন**
> webman ভার্সন 1.4.6 এর আগে কন্ট্রোলার প্যারামিটার ইনজেকশন সাপোর্ট করে না। যেমন নিচের কোড webman<=1.4.6 এ সাপোর্টেড নয়:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // 1.4.6 এর আগে কন্ট্রোলার প্যারামিটার ইনজেকশন সাপোর্ট করে না
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

## কাস্টম কনস্ট্রাক্টর ইনজেকশন

কখনও কখনও কনস্ট্রাক্টরে পাঠানো প্যারামিটার ক্লাস ইনস্ট্যান্স না হয়ে স্ট্রিং, সংখ্যা, অ্যারে ইত্যাদি হতে পারে। যেমন Mailer কনস্ট্রাক্টরে SMTP সার্ভার IP ও পোর্ট লাগতে পারে:

```php
<?php
namespace app\service;

class Mailer
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // ইমেইল পাঠানোর কোড বাদ দেওয়া হয়েছে
    }
}
```

এ অবস্থায় আগের কনস্ট্রাক্টর অটো ইনজেকশন সরাসরি ব্যবহার করা যায় না কারণ `php-di` জানে না `$smtp_host` ও `$smtp_port` এর মান কী। এ ক্ষেত্রে কাস্টম ইনজেকশন ব্যবহার করা যায়।

`config/dependence.php` এ (ফাইল না থাকলে তৈরি করুন) নিচের কোড যোগ করুন:

```php
return [
    // ... অন্যান্য কনফিগারেশন বাদ দেওয়া হয়েছে

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

ডিপেন্ডেন্সি ইনজেকশন যখন `app\service\Mailer` ইনস্ট্যান্স চাইবে তখন এই কনফিগে তৈরি ইনস্ট্যান্স অটোমেটিক ব্যবহার করবে।

লক্ষ্য করুন `config/dependence.php` এ `Mailer` ক্লাস ইনস্ট্যান্স করতে `new` ব্যবহার হচ্ছে। এ উদাহরণে কোন সমস্যা নেই কিন্তু `Mailer` ক্লাস যদি অন্য ক্লাসে ডিপেন্ড করে বা ভিতরে অ্যাট্রিবিউট ইনজেকশন ব্যবহার করে তাহলে `new` দিয়ে ইনিশিয়ালাইজ করলে ডিপেন্ডেন্সি অটো ইনজেকশন হবে না। সমাধান হল কাস্টম ইন্টারফেস ইনজেকশন এবং `Container::get(ক্লাস নাম)` বা `Container::make(ক্লাস নাম, [কনস্ট্রাক্টর প্যারামিটার])` দিয়ে ক্লাস ইনিশিয়ালাইজ করা।

## কাস্টম ইন্টারফেস ইনজেকশন

প্রকৃত প্রজেক্টে কংক্রিট ক্লাসের বদলে ইন্টারফেসে প্রোগ্রাম করা ভালো। যেমন `app\controller\UserController` কে `app\service\Mailer` এর বদলে `app\service\MailerInterface` এ ডিপেন্ড করা উচিত।

`MailerInterface` ইন্টারফেস ডিফাইন করুন:

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

`MailerInterface` এর ইমপ্লিমেন্টেশন ডিফাইন করুন:

```php
<?php
namespace app\service;

class Mailer implements MailerInterface
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // ইমেইল পাঠানোর কোড বাদ দেওয়া হয়েছে
    }
}
```

কংক্রিট ইমপ্লিমেন্টেশন এর বদলে `MailerInterface` ব্যবহার করুন:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\MailerInterface;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private MailerInterface $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

`config/dependence.php` এ `MailerInterface` এর ইমপ্লিমেন্টেশন ডিফাইন করুন:

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

বিজনেস যখন `MailerInterface` ইন্টারফেস ব্যবহার করবে তখন অটোমেটিক `Mailer` ইমপ্লিমেন্টেশন ব্যবহার হবে।

> ইন্টারফেসে প্রোগ্রাম করার সুবিধা হল কম্পোনেন্ট বদলাতে হলে বিজনেস কোড বদলাতে হয় না — শুধু `config/dependence.php` এ কংক্রিট ইমপ্লিমেন্টেশন বদলাতে হয়। ইউনিট টেস্টের জন্যও খুব কাজের।

## অন্যান্য কাস্টম ইনজেকশন

ক্লাস ডিপেন্ডেন্সি ছাড়াও `config/dependence.php` এ স্ট্রিং, সংখ্যা, অ্যারে ইত্যাদি অন্যান্য ভ্যালু ডিফাইন করা যায়।

যেমন `config/dependence.php` যদি এভাবে ডিফাইন হয়:

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

তখন `#[Inject]` দিয়ে `smtp_host` ও `smtp_port` ক্লাস প্রোপার্টিতে ইনজেক্ট করা যায়:

```php
<?php
namespace app\service;

use DI\Attribute\Inject;

class Mailer
{
    #[Inject("smtp_host")]
    private $smtpHost;

    #[Inject("smtp_port")]
    private $smtpPort;

    public function mail($email, $content)
    {
        // ইমেইল পাঠানোর কোড বাদ দেওয়া হয়েছে
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // 192.168.1.11:25 আউটপুট হবে
    }
}
```

# লেজি লোডিং

> লেজি লোডিং একটি ডিজাইন প্যাটার্ন যা অবজেক্ট তৈরি বা ইনিশিয়ালাইজ করা পর্যন্ত পিছিয়ে দেয় যতক্ষণ না সত্যিই প্রয়োজন হয়।

এ ফিচারের জন্য অতিরিক্ত ডিপেন্ডেন্সি লাগে। নিচের প্যাকেজটি `ocramius/proxy-manager` এর একটি ফোর্ক; মূল রিপোজিটরি PHP 8 সাপোর্ট করে না।

```
composer require friendsofphp/proxy-manager-lts
```

ব্যবহার:

```php
<?php

use DI\Attribute\Injectable;
use DI\Attribute\Inject;

#[Injectable(lazy: true)]
class MyClass
{
    private string $name;

    public function __construct()
    {
        echo "MyClass ইনস্ট্যান্স হয়েছে\n";
        $this->name = "Lazy Loaded Object";
    }

    public function getName(): string
    {
        return $this->name;
    }
}

class Controller
{
    #[Inject]
    public MyClass $myClass;

    public function getClass()
    {
        echo "প্রক্সি ক্লাস নাম: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

আউটপুট:

```
প্রক্সি ক্লাস নাম: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass ইনস্ট্যান্স হয়েছে
name: Lazy Loaded Object
```

এ উদাহরণ দেখায় যে `#[Injectable]` অ্যাট্রিবিউট সহ ডিক্লেয়ারড ক্লাস ইনজেক্ট হলে আগে তার প্রক্সি ক্লাস তৈরি হয়। আসল ক্লাস শুধু তার যেকোনো মেথড কল হলেই ইনস্ট্যান্স হয়।

# সিরকুলার ডিপেন্ডেন্সি

সিরকুলার ডিপেন্ডেন্সি হয় যখন একাধিক ক্লাস একে অপরের উপর নির্ভর করে একটি বন্ধ ডিপেন্ডেন্সি লুপ তৈরি করে।

- ডাইরেক্ট সিরকুলার ডিপেন্ডেন্সি
  - মডিউল A মডিউল B তে ডিপেন্ড করে, মডিউল B মডিউল A তে ডিপেন্ড করে
  - A → B → A লুপ তৈরি হয়

- ইন্ডাইরেক্ট সিরকুলার ডিপেন্ডেন্সি
  - ডিপেন্ডেন্সি সাইকেলে একাধিক মডিউল জড়িত
  - যেমন A → B → C → A

অ্যাট্রিবিউট ইনজেকশন ব্যবহার করলে `php-di` সিরকুলার ডিপেন্ডেন্সি অটো ডিটেক্ট করে এক্সেপশন দেয়। দরকার হলে নিচের পদ্ধতি ব্যবহার করুন:

```php
class userController
{

    // এ কোড মুছে দিন
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## আরও তথ্য

দয়া করে [php-di ডকুমেন্টেশন](https://php-di.org/doc/getting-started.html) দেখুন।
