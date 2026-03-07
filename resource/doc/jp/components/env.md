# ENVコンポーネント vlucas/phpdotenv

## 説明
`vlucas/phpdotenv`は、異なる環境（開発環境、テスト環境など）の設定を区別するための環境変数ロードコンポーネントです。

## プロジェクトアドレス

https://github.com/vlucas/phpdotenv
  
## インストール
 
```php
composer require vlucas/phpdotenv
 ```
  
## 使用方法

### プロジェクトのルートディレクトリに`.env`ファイルを新規作成
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### 設定ファイルを変更
**config/database.php**
```php
return [
    // デフォルトデータベース
    'default' => 'mysql',

    // 各種データベースの設定
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **ヒント**
> `.env`ファイルを`.gitignore`に追加し、リポジトリにコミットしないことをお勧めします。リポジトリには`.env.example`という設定サンプルを用意し、デプロイ時は`.env.example`を`.env`にコピーして、環境に合わせて設定を変更してください。これにより、環境ごとに異なる設定を読み込めます。

> **注意**
> `vlucas/phpdotenv`はPHPのTS版（スレッドセーフ版）でバグが発生する可能性があります。NTS版（非スレッドセーフ版）をご利用ください。現在のPHPバージョンは`php -v`で確認できます。

## 詳細

https://github.com/vlucas/phpdotenv をご覧ください。
  
