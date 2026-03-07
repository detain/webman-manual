# Medoo データベース

[webman/medoo](https://github.com/webman-php/medoo)は[Medoo](https://medoo.in/)にコネクションプールを追加し、コルーチン環境と非コルーチン環境の両方で動作します。使い方はMedooと同じです。

## インストール
`composer require webman/medoo`

## Medoo データベース設定
設定ファイルの場所：`config/plugin/webman/medoo/database.php`

## Medoo データベースの使い方
```php
<?php
namespace app\controller;

use support\Request;
use support\Medoo;

class Index
{
    public function index(Request $request)
    {
        $user = Medoo::get('user', '*', ['uid' => 1]);
        return json($user);
    }
}
```

> **ヒント**
> `Medoo::get('user', '*', ['uid' => 1]);`
> は
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`
> と同等です。

## Medoo 複数データベース設定

**設定**
`config/plugin/webman/medoo/database.php` に新しい設定を追加します。キーは任意で、ここでは `other` を使用しています。

```php
<?php
return [
    'default' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [ // コネクションプール設定
            'max_connections' => 5, // 最大接続数
            'min_connections' => 1, // 最小接続数
            'wait_timeout' => 60,   // プールから接続を取得する際の最大待ち時間、超えると例外が発生
            'idle_timeout' => 3,    // プール内接続の最大アイドル時間、超えると回収され最小接続数まで減る
            'heartbeat_interval' => 50, // プールの heartbeat 間隔（秒）、60秒未満を推奨
        ]
    ],
    // ここに other の設定を追加
    'other' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [
            'max_connections' => 5,
            'min_connections' => 1,
            'wait_timeout' => 60,
            'idle_timeout' => 3,
            'heartbeat_interval' => 50,
        ],
    ],
];
```

## Medoo データベースの使い方
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

[Medoo 公式ドキュメント](https://medoo.in/api/select) を参照してください。
