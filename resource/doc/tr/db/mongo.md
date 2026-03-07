# MongoDB

webman varsayılan olarak [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb)'ı MongoDB bileşeni olarak kullanır. Bu, Laravel projesinden çıkarılmıştır ve Laravel ile aynı şekilde çalışır.

`mongodb/laravel-mongodb` kullanmadan önce `php-cli` için MongoDB eklentisini yüklemeniz gerekir.

> **Not**
> `php-cli` için MongoDB eklentisinin yüklü olup olmadığını kontrol etmek için `php -m | grep mongodb` komutunu kullanın. Not: `php-fpm` için MongoDB eklentisini yüklemiş olsanız bile, bu `php-cli`'da kullanabileceğiniz anlamına gelmez, çünkü `php-cli` ve `php-fpm` farklı uygulamalardır ve farklı `php.ini` yapılandırmalarını kullanabilirler. Kullandığınız `php-cli`'ın hangi `php.ini` yapılandırma dosyasını kullandığını görmek için `php --ini` komutunu kullanın.

## Kurulum

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

Kurulumdan sonra restart yapılması gerekmektedir (reload yeterli değildir).

## Yapılandırma
`config/database.php` dosyasına aşağıdaki gibi `mongodb` bağlantısını ekleyin:
```php
return [

    'default' => 'mysql',

    'connections' => [

         ...diğer yapılandırmalar burada atlandı...

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => '127.0.0.1',
            'port'     =>  27017,
            'database' => 'test',
            'username' => null,
            'password' => null,
            'options' => [
                // burada Mongo Driver Manager'a daha fazla ayar gönderebilirsiniz
                // kullanabileceğiniz tam parametre listesi için https://www.php.net/manual/en/mongodb-driver-manager.construct.php adresine bakın

                'appname' => 'homestead'
            ],
        ],
    ],
];
```

## Örnek
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

## Model Örneği
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

## Daha fazla bilgi için ziyaret edin

https://github.com/mongodb/laravel-mongodb
