# MongoDB

webman 預設使用 [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) 作為 MongoDB 組件，它是從 Laravel 專案中抽離出來的，用法與 Laravel 相同。

在使用 `mongodb/laravel-mongodb` 之前，必須先給 `php-cli` 安裝 MongoDB 擴展。

> **注意**
> 使用命令 `php -m | grep mongodb` 檢查 `php-cli` 是否安裝了 MongoDB 擴展。注意：即使你在 `php-fpm` 安裝了 MongoDB 擴展，不代表你在 `php-cli` 可以使用它，因為 `php-cli` 和 `php-fpm` 是不同的應用程式，可能使用的是不同的 `php.ini` 配置。使用命令 `php --ini` 來查看你的 `php-cli` 使用的是哪個 `php.ini` 配置檔案。

## 安裝

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

安裝後需要 restart 重啟（reload 無效）

## 配置
在 `config/database.php` 裡增加 `mongodb` connection，類似如下：
```php
return [

    'default' => 'mysql',

    'connections' => [

         ...這裡省略了其他配置...

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

## 示例
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

## 模型示例
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

## 更多內容請訪問

https://github.com/mongodb/laravel-mongodb
