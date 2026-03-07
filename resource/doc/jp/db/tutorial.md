# データベースクイックスタート（Laravelデータベースコンポーネントベース）

[webman/database](https://github.com/webman-php/database)は[illuminate/database](https://github.com/illuminate/database)を基に開発され、コネクションプール機能を追加。コルーチン環境と非コルーチン環境の両方をサポートし、Laravelと同じ使い方です。

[他のデータベースコンポーネントを使用する](others.md)の章を参照して、ThinkPHPや他のデータベースを使うこともできます。

## データベースのインストール

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

インストール後はrestartで再起動が必要です（reloadでは反映されません）。

> **ヒント**
> webman/databaseはLaravelの`illuminate/database`に依存するため、インストール時に`illuminate/database`の依存パッケージも自動でインストールされます。

> **注意**
> ページネーション、データベースイベント、SQLログが不要な場合は、次のコマンドだけを実行してください。
> `composer require -W webman/database`

## データベースの設定
`config/database.php`
```php

return [
    // デフォルトデータベース
    'default' => 'mysql',

    // 各種データベース接続の設定
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'test',
            'username'    => 'root',
            'password'    => '',
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
            'options' => [
                PDO::ATTR_EMULATE_PREPARES => false, // swooleまたはswowをドライバーにする場合は必須
            ],
            'pool' => [ // コネクションプール設定
                'max_connections' => 5, // 最大接続数
                'min_connections' => 1, // 最小接続数
                'wait_timeout' => 3,    // プールから接続を取得する最大待機時間。タイムアウトで例外。コルーチン環境のみ有効
                'idle_timeout' => 60,   // プール内接続の最大アイドル時間。タイムアウトで回収され、min_connectionsまで減る
                'heartbeat_interval' => 50, // プールのハートビート間隔（秒）。60秒未満推奨
            ],
        ],
    ],
];
```

`pool`設定以外は、Laravelと同じです。

## コネクションプールについて
* 各プロセスが独自のコネクションプールを持ち、プロセス間では共有されません。
* コルーチンを有効にしていない場合、リクエストはプロセス内で順次実行され、並行性がないため、プールの接続は最大1つだけです。
* コルーチン有効時は、リクエストがプロセス内で並行実行され、プールは需要に応じて接続数を動的に調整します。`max_connections`を超えず、`min_connections`を下回りません。
* プールの接続数は最大`max_connections`のため、データベースを操作するコルーチン数がこれを超えると、待機列で最大`wait_timeout`秒待ち、超過すると例外が発生します。
* アイドル時（コルーチン・非コルーチン両方）は、接続は`idle_timeout`経過後に回収され、接続数が`min_connections`になるまで続きます（`min_connections`は0可）。


## データベース使用例
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        $default_uid = 29;
        $uid = $request->get('uid', $default_uid);
        $name = Db::table('users')->where('uid', $uid)->value('username');
        return response("hello $name");
    }
}
```

Laravelと同じく、`Db::table()`メソッドでデータベースを操作します。
