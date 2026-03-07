# MongoDB

webman mặc định sử dụng [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) làm thành phần MongoDB. Thành phần này được trích xuất từ dự án Laravel và hoạt động tương tự như Laravel.

Trước khi sử dụng `mongodb/laravel-mongodb`, bạn cần cài đặt extension MongoDB cho `php-cli`.

> **Lưu ý**
> Sử dụng lệnh `php -m | grep mongodb` để kiểm tra xem `php-cli` đã cài đặt extension MongoDB hay chưa. Lưu ý: Ngay cả khi bạn đã cài đặt extension MongoDB cho `php-fpm`, điều này không đảm bảo rằng bạn có thể sử dụng nó trong `php-cli` vì `php-cli` và `php-fpm` là hai ứng dụng khác nhau và có thể sử dụng cấu hình `php.ini` khác nhau. Sử dụng lệnh `php --ini` để xem file cấu hình `php.ini` nào được sử dụng bởi `php-cli`.

## Cài đặt

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

Sau khi cài đặt, bạn cần restart (reload không đủ).

## Cấu hình
Trong tập tin `config/database.php`, thêm kết nối `mongodb`, tương tự như sau:
```php
return [

    'default' => 'mysql',

    'connections' => [

         ...các cấu hình khác được bỏ qua ở đây...

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => '127.0.0.1',
            'port'     =>  27017,
            'database' => 'test',
            'username' => null,
            'password' => null,
            'options' => [
                // ở đây bạn có thể truyền thêm cài đặt cho Mongo Driver Manager
                // xem chi tiết các tham số mà bạn có thể sử dụng tại https://www.php.net/manual/en/mongodb-driver-manager.construct.php trong phần "Uri Options"

                'appname' => 'homestead'
            ],
        ],
    ],
];
```

## Ví dụ
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

## Ví dụ Model
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

## Xem thêm tại

https://github.com/mongodb/laravel-mongodb
