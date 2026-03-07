# MongoDB

webman डिफ़ॉल्ट रूप से [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) का उपयोग MongoDB कंपोनेंट के रूप में करता है। यह Laravel प्रोजेक्ट से निकाला गया है और Laravel के समान उपयोग किया जाता है।

`mongodb/laravel-mongodb` का उपयोग करने से पहले, `php-cli` में MongoDB extension को स्थापित करना आवश्यक है।

> **ध्यान दें**
> `php -m | grep mongodb` कमांड का उपयोग करके देखें कि `php-cli` में MongoDB extension स्थापित है या नहीं। ध्यान दें: यहां तक कि अगर आपने `php-fpm` में MongoDB extension स्थापित कर लिया है, तो यह नहीं संकेत देता है कि आप `php-cli` में इसका उपयोग कर सकते हैं, क्योंकि `php-cli` और `php-fpm` अलग-अलग अनुप्रयोग हैं और विभिन्न `php.ini` कॉन्फ़िगरेशन का उपयोग कर सकते हैं। अपनी `php-cli` द्वारा किस `php.ini` कॉन्फ़िगरेशन फ़ाइल का उपयोग हो रहा है, उसे देखने के लिए `php --ini` कमांड का उपयोग करें।

## स्थापना

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

स्थापना के बाद restart की जरूरत होती है (reload पर्याप्त नहीं है)

## कॉन्फ़िगरेशन
`config/database.php` में `mongodb` कनेक्शन जोड़ें, निम्नलिखित तरह से:

```php
return [

    'default' => 'mysql',

    'connections' => [

         ...अन्य कॉन्फ़िगरेशन यहां छोड़ दी गई है...

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

## उदाहरण
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

## मॉडल उदाहरण
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

## अधिक जानकारी के लिए यहां जाएं

https://github.com/mongodb/laravel-mongodb
