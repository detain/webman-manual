# MongoDB

webman পূর্বনির্ধারিতভাবে [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) ব্যবহার করে, যা Laravel প্রকল্প থেকে আলাদা করা হয়েছে, Laravel এর মত ব্যবহার করা হয়।

`mongodb/laravel-mongodb` ব্যবহার করার আগে প্রথমে `php-cli` তে MongoDB এক্সটেনশন ইনস্টল করতে হবে।

> **লক্ষ্য করুন**
> `php-cli` এ MongoDB এক্সটেনশন ইনস্টল আছে কিনা দেখতে `php -m | grep mongodb` কমান্ড ব্যবহার করুন। লক্ষ্য করুন: যদি আপনি `php-fpm` এ MongoDB এক্সটেনশন ইনস্টল করে থাকেন, তবে এটা দর্শায় না যে আপনি `php-cli` তে এটি ব্যবহার করতে পারবেন, কারণ `php-cli` এবং `php-fpm` দুটি ভিন্ন অ্যাপ্লিকেশন, এরা ভিন্ন `php.ini` কনফিগারেশন ব্যবহার করতে পারে। আপনার `php-cli` কোন `php.ini` কনফিগারেশন ফাইল ব্যবহার করছে তা দেখতে `php --ini` কমান্ড ব্যবহার করুন।

## ইনস্টলেশন

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

ইনস্টলেশন পরে restart করতে হবে (reload করলে কাজ হবে না)

## কনফিগারেশন
`config/database.php` ফাইলে `mongodb` কানেকশন যোগ করুন, নিম্নলিখিত মত।
```php
return [

    'default' => 'mysql',

    'connections' => [

         ... অন্যান্য কনফিগারেশন এখানে অপসারণ করা হয়েছে ...

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => '127.0.0.1',
            'port'     =>  27017,
            'database' => 'test',
            'username' => null,
            'password' => null,
            'options' => [
                // here you can pass more settings to the Mongo Driver Manager
                // see https://www.php.net/manual/en/mongodb-driver-manager.construct.php under "Uri Options" for a list of complete parameters that you can use

                'appname' => 'homestead'
            ],
        ],
    ],
];
```

## নমূনা
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        Db::connection('mongodb')->table('test')->insert([1,2,3]);
        return json(Db::connection('mongodb')->table('test')->get());
    }
}
```

## মডেল নমূনা
```php
<?php
namespace app\model;

use DateTimeInterface;
use support\MongoModel as Model;

class Test extends Model
{
    protected $connection = 'mongodb';

    protected $table = 'test';

    public $timestamps = true;

    /**
     * @param DateTimeInterface $date
     * @return string
     */
    protected function serializeDate(DateTimeInterface $date): string
    {
        return $date->format('Y-m-d H:i:s');
    }
}
```

## অধিক তথ্য জানতে ভিজিট করুন

https://github.com/mongodb/laravel-mongodb
