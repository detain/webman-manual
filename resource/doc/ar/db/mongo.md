# MongoDB

يستخدم webman افتراضيًا [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) كمكون MongoDB، وهو مُستخرج من مشروع Laravel ويعمل بنفس طريقة Laravel.

يجب تثبيت امتداد MongoDB لـ `php-cli` قبل استخدام `mongodb/laravel-mongodb`.

> **ملاحظة**
> استخدم الأمر `php -m | grep mongodb` للتحقق مما إذا كان امتداد mongodb مثبتًا لـ `php-cli`. انتبه: حتى لو قمت بتثبيت امتداد mongodb لـ `php-fpm`، فهذا لا يعني أنك يمكنك استخدامه في `php-cli`، لأن `php-cli` و `php-fpm` هما تطبيقان مختلفان وقد يستخدمان تكوينات `php.ini` مختلفة. استخدم الأمر `php --ini` لمعرفة ملف تكوين `php.ini` الذي يستخدمه `php-cli`.

## التثبيت

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

بعد التثبيت يلزم إعادة التشغيل (reload غير كافٍ).

## التكوين
أضف اتصال `mongodb` في `config/database.php`، كما يلي:
```php
return [

    'default' => 'mysql',

    'connections' => [

         ...إعدادات أخرى محذوفة هنا...

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => '127.0.0.1',
            'port'     =>  27017,
            'database' => 'test',
            'username' => null,
            'password' => null,
            'options' => [
                // هنا يمكنك تمرير المزيد من الإعدادات إلى Mongo Driver Manager
                // راجع https://www.php.net/manual/en/mongodb-driver-manager.construct.php تحت "Uri Options" للحصول على قائمة بالمعاملات الكاملة التي يمكنك استخدامها

                'appname' => 'homestead'
            ],
        ],
    ],
];
```

## مثال
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

## مثال على النموذج
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

## لمزيد من المعلومات يرجى زيارة

https://github.com/mongodb/laravel-mongodb
