# ডেটাবেস

যেহেতু বেশিরভাগ প্লাগইন [webman-admin](https://www.workerman.net/plugin/82) ইন্সটল করে, তাই `webman-admin`-এর ডেটাবেস কনফিগারেশন সরাসরি পুনরায় ব্যবহার করার পরামর্শ দেওয়া হয়।

যেসব মডেলের বেস ক্লাস `plugin\admin\app\model\Base`, তারা স্বয়ংক্রিয়ভাবে webman-admin-এর ডেটাবেস কনফিগারেশন ব্যবহার করবে।
```php
<?php

namespace plugin\foo\app\model;

use plugin\admin\app\model\Base;

class Orders extends Base
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'foo_orders';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $primaryKey = 'id';
    
}
```

`plugin.admin.mysql` এর মাধ্যমেও webman-admin ডেটাবেসে অ্যাক্সেস করা যায়, উদাহরণস্বরূপ:

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## নিজের ডেটাবেস ব্যবহার করা

প্লাগইন নিজের ডেটাবেস ব্যবহার করতেও বেছে নিতে পারে। উদাহরণস্বরূপ, `plugin/foo/config/database.php` এর বিষয়বস্তু:

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql কানেকশনের নাম
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'ডেটাবেস',
            'username'    => 'ব্যবহারকারী_নাম',
            'password'    => 'পাসওয়ার্ড',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin কানেকশনের নাম
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'ডেটাবেস',
            'username'    => 'ব্যবহারকারী_নাম',
            'password'    => 'পাসওয়ার্ড',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

রেফারেন্স ফরম্যাট হল `Db::connection('plugin.{প্লাগইন}.{কানেকশন_নাম}');`, উদাহরণস্বরূপ:

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

মূল প্রজেক্টের ডেটাবেস ব্যবহার করতে সরাসরি কল করুন:

```php
use support\Db;
Db::table('user')->first();
// ধরে নিন মূল প্রজেক্টে admin কানেকশনও কনফিগার করা আছে
Db::connection('admin')->table('admin')->first();
```

#### Model এর জন্য ডেটাবেস কনফিগার করা

Model এর জন্য একটি Base ক্লাস তৈরি করতে পারেন এবং প্লাগইনের নিজস্ব ডেটাবেস কানেকশন ব্যবহার করার জন্য `$connection` প্রপার্টি সেট করতে পারেন:

```php
<?php

namespace plugin\foo\app\model;

use DateTimeInterface;
use support\Model;

class Base extends Model
{
    /**
     * @var string
     */
    protected $connection = 'plugin.foo.mysql';

}
```

এভাবে প্লাগইনের সব মডেল Base থেকে ইনহেরিট করলে স্বয়ংক্রিয়ভাবে প্লাগইনের নিজস্ব ডেটাবেস ব্যবহার করবে।

## স্বয়ংক্রিয় ডেটাবেস ইম্পোর্ট
`php webman app-plugin:create foo` চালানোর ফলে foo প্লাগইন স্বয়ংক্রিয়ভাবে তৈরি হয়, যাতে `plugin/foo/api/Install.php` এবং `plugin/foo/install.sql` অন্তর্ভুক্ত থাকে।

> **পরামর্শ**
> install.sql ফাইল জেনারেট না হলে `webman/console` আপগ্রেড করার চেষ্টা করুন: `composer require webman/console ^1.3.6`

#### প্লাগইন ইন্সটল করার সময় ডেটাবেস ইম্পোর্ট করা
প্লাগইন ইন্সটল হলে Install.php এর `install` মেথড চালু হয়, যা `install.sql` এর SQL স্টেটমেন্টগুলো স্বয়ংক্রিয়ভাবে এক্সিকিউট করে, ফলে ডেটাবেস টেবিল ইম্পোর্ট হয়।

`install.sql` এর বিষয়বস্তু টেবিল তৈরির এবং টেবিলের ঐতিহাসিক পরিবর্তনের SQL স্টেটমেন্ট হওয়া উচিত। প্রতিটি স্টেটমেন্ট `;` দিয়ে শেষ হতে হবে, উদাহরণস্বরূপ:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'প্রাথমিক কী',
  `order_id` varchar(50) NOT NULL COMMENT 'অর্ডার আইডি',
  `user_id` int NOT NULL COMMENT 'ব্যবহারকারী আইডি',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'পরিশোধের পরিমাণ',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='অর্ডার';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'প্রাথমিক কী',
  `name` varchar(50) NOT NULL COMMENT 'নাম',
  `price` int NOT NULL COMMENT 'দাম',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='পণ্য';
```

**ডেটাবেস কানেকশন পরিবর্তন করা**

ডিফল্টভাবে `install.sql` webman-admin-এর ডেটাবেসে ইম্পোর্ট হয়। অন্য ডেটাবেসে ইম্পোর্ট করতে Install.php-এ `$connection` প্রপার্টি পরিবর্তন করুন:

```php
<?php

class Install
{
    // প্লাগইনের নিজস্ব ডেটাবেস নির্দিষ্ট করুন
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**পরীক্ষা**

প্লাগইন ইন্সটল করতে `php webman app-plugin:install foo` চালান। তারপর ডেটাবেস চেক করুন — `foo_orders` এবং `foo_goods` টেবিল তৈরি হয়ে থাকবে।

#### প্লাগইন আপগ্রেডের সময় টেবিল স্ট্রাকচার পরিবর্তন করা
কখনও কখনও প্লাগইন আপগ্রেডে নতুন টেবিল বা টেবিল স্ট্রাকচার পরিবর্তন প্রয়োজন হয়, `install.sql` এর শেষে সংশ্লিষ্ট স্টেটমেন্ট যোগ করতে পারেন। প্রতিটি স্টেটমেন্ট `;` দিয়ে শেষ হতে হবে। উদাহরণস্বরূপ, `foo_user` টেবিল যোগ করা এবং `foo_orders` টেবিলে `status` কলাম যোগ করা
```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'প্রাথমিক কী',
  `order_id` varchar(50) NOT NULL COMMENT 'অর্ডার আইডি',
  `user_id` int NOT NULL COMMENT 'ব্যবহারকারী আইডি',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'পরিশোধের পরিমাণ',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='অর্ডার';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'প্রাথমিক কী',
 `name` varchar(50) NOT NULL COMMENT 'নাম',
 `price` int NOT NULL COMMENT 'দাম',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='পণ্য';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'প্রাথমিক কী',
 `name` varchar(50) NOT NULL COMMENT 'নাম'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='ব্যবহারকারী';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT 'অবস্থা';
```

আপগ্রেডের সময় Install.php এর `update` মেথড চালু হয়, যা `install.sql` এর স্টেটমেন্টগুলো এক্সিকিউট করে। নতুন স্টেটমেন্ট থাকলে তা এক্সিকিউট হয়, পুরনো স্টেটমেন্ট স্কিপ হয়ে যায়, ফলে আপগ্রেডে ডেটাবেস পরিবর্তন সঠিকভাবে প্রয়োগ হয়।

#### প্লাগইন আনইন্সটল করার সময় ডেটাবেস মুছা
প্লাগইন আনইন্সটল হলে Install.php এর `uninstall` মেথড কল হয়। এটি `install.sql` এর CREATE TABLE স্টেটমেন্টগুলো স্বয়ংক্রিয়ভাবে বিশ্লেষণ করে এবং সেই টেবিলগুলো মুছে দেয়।
আনইন্সটল করার সময় শুধুমাত্র নিজের `uninstall.sql` এক্সিকিউট করতে চাইলে, স্বয়ংক্রিয় টেবিল মুছার অপারেশন না করতে চাইলে, শুধু `plugin/প্লাগইন_নাম/uninstall.sql` তৈরি করুন। এভাবে `uninstall` মেথড শুধুমাত্র `uninstall.sql` ফাইলে স্টেটমেন্টগুলো এক্সিকিউট করবে।
