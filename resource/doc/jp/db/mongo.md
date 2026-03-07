# MongoDB

webmanはデフォルトで [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) をMongoDBコンポーネントとして使用しています。これはLaravelプロジェクトから抽出されたもので、Laravelと同様の使い方ができます。

`mongodb/laravel-mongodb` を使用する前に、`php-cli` にMongoDB拡張をインストールする必要があります。

> **注意**
> `php -m | grep mongodb` コマンドを使用して、`php-cli` にMongoDB拡張がインストールされているか確認してください。注意：`php-fpm` にMongoDB拡張をインストールしていても、`php-cli` で使用できるとは限りません。`php-cli` と `php-fpm` は異なるアプリケーションであり、異なる `php.ini` 構成を使用している可能性があるためです。`php-cli` がどの `php.ini` 設定ファイルを使用しているか確認するには、`php --ini` コマンドを使用してください。

## インストール

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

インストール後は再起動が必要です（reloadでは無効です）。

## 設定
`config/database.php` に `mongodb` 接続を追加します。以下は例です：
```php
return [

    'default' => 'mysql',

    'connections' => [

         ...他の設定は省略...

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => '127.0.0.1',
            'port'     =>  27017,
            'database' => 'test',
            'username' => null,
            'password' => null,
            'options' => [
                // ここではMongo Driver Managerにさらなる設定を渡すことができます
                // 使用可能な完全なパラメータのリストについては、https://www.php.net/manual/en/mongodb-driver-manager.construct.php の「Uri Options」を参照してください

                'appname' => 'homestead'
            ],
        ],
    ],
];
```

## 例
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

## モデル例
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

## 詳細は以下をご覧ください

https://github.com/mongodb/laravel-mongodb
