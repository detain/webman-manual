# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) হল [top-think/think-orm](https://github.com/top-think/think-orm) এর উপর ভিত্তি করে বিকশিত একটি ডেটাবেস উপাদান। এটি কানেকশন পুল সমর্থন করে এবং করউটিন ও নন-করউটিন উভয় পরিবেশে কাজ করে।

## ইনস্টলেশন

`composer require -W webman/think-orm`

ইনস্টল হওয়ার পর restart (পুনরায় চালু) প্রয়োজন (reload কার্যকর হয় না)।

## কনফিগারেশন ফাইল

আপনার প্রকৃত প্রয়োজন অনুযায়ী `config/think-orm.php` কনফিগারেশন ফাইল সম্পাদনা করুন।

## নথিপত্র

https://www.kancloud.cn/manual/think-orm

## ব্যবহার

```php
<?php
namespace app\controller;

use support\Request;
use support\think\Db;

class FooController
{
    public function get(Request $request)
    {
        $user = Db::table('user')->where('uid', '>', 1)->find();
        return json($user);
    }
}
```

## মডেল তৈরি

think-orm মডেলগুলি `support\think\Model` উত্তরাধিকারসূত্রে প্রাপ্ত, নিচের মতো:

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $pk = 'id';

}
```

আপনি নিচের কমান্ড দিয়ে think-orm মডেলও তৈরি করতে পারেন:

```
php webman make:model টেবিলের_নাম
```

> **পরামর্শ**
> এই কমান্ডের জন্য `webman/console` প্রয়োজন। ইনস্টল করুন: `composer require webman/console ^1.2.13`

> **লক্ষ্য করুন**
> যদি make:model কমান্ডটি বুঝতে পারে যে মূল প্রজেক্ট `illuminate/database` ব্যবহার করছে, তাহলে think-orm এর পরিবর্তে Illuminate ভিত্তিক মডেল ফাইল তৈরি হবে। সে ক্ষেত্রে `tp` প্যারামিটার যোগ করুন: `php webman make:model টেবিলের_নাম tp` (কাজ না করলে `webman/console` আপডেট করুন)
