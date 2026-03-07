# イベント処理
`webman/event`は、コードを変更することなくビジネスロジックを実行できる精巧なイベント機構を提供し、モジュール間の結合を緩和します。典型的なシナリオ：新規ユーザーが登録成功した際、`user.register`のようなカスタムイベントを発行するだけで、各モジュールがそのイベントを受け取り、対応するビジネスロジックを実行できます。

## インストール
`composer require webman/event`

## イベントの購読
イベント購読は`config/event.php`で統一して設定します。

```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...その他のイベント処理関数...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...その他のイベント処理関数...
    ]
];
```

**説明：**
- `user.register`、`user.logout`などはイベント名（文字列型）です。小文字でドット（`.`）区切りを推奨します。
- 1つのイベントに複数の処理関数を紐づけられ、設定順に呼び出されます。

## イベント処理関数
処理関数はクラスメソッド、関数、クロージャのいずれでも可能です。

例：`app/event/User.php`を作成（ディレクトリが無い場合は自動作成）

```php
<?php
namespace app\event;
class User
{
    function register($user)
    {
        var_export($user);
    }
 
    function logout($user)
    {
        var_export($user);
    }
}
```

## イベントの発行
`Event::dispatch($event_name, $data);` または `Event::emit($event_name, $data);` でイベントを発行します。例：

```php
<?php
namespace app\controller;
use support\Request;
use Webman\Event\Event;
class User
{
    public function register(Request $request)
    {
        $user = [
            'name' => 'webman',
            'age' => 2
        ];
        Event::dispatch('user.register', $user);
    }
}
```

発行には`Event::dispatch($event_name, $data);`と`Event::emit($event_name, $data);`の2つがあり、引数は同じです。違いは、`emit`は内部で例外を捕捉するため、1つのイベントに複数の処理関数がある場合、ある関数で例外が発生しても他は実行されます。一方、`dispatch`は例外を捕捉しないため、いずれかの処理関数で例外が発生すると、以降の処理は止まり例外が上位に伝播します。

> **ヒント**
> パラメータ`$data`は配列、クラスインスタンス、文字列など任意のデータを渡せます。

## ワイルドカードイベントリスナー
ワイルドカード登録により、同じリスナーで複数イベントを扱えます。例：`config/event.php`で

```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```

処理関数の第2引数`$event_data`で具体的なイベント名を取得できます：

```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // 具体的なイベント名（user.register、user.logout など）
        var_export($user);
    }
}
```

## イベントブロードキャストの停止
処理関数内で`false`を返すと、そのイベントのブロードキャストは停止します。

## クロージャでのイベント処理
処理関数はクラスメソッドでもクロージャでも構いません。例：

```php
<?php
return [
    'user.login' => [
        function($user){
            var_dump($user);
        }
    ]
];
```

## イベントとリスナーの表示
`php webman event:list`コマンドで、プロジェクト内で設定されたイベントとリスナーを表示できます。

## 対応範囲
メインプロジェクトに加え、[ベースプラグイン](../plugin/base.md)および[アプリプラグイン](../app/app.md)も`event.php`設定に対応しています。
**ベースプラグイン設定ファイル：** `config/plugin/ベンダー名/プラグイン名/event.php`
**アプリプラグイン設定ファイル：** `plugin/プラグイン名/config/event.php`

## 注意事項
イベント処理は非同期ではありません。遅い処理には向いておらず、遅いロジックは[webman/redis-queue](https://www.workerman.net/plugin/12)などのメッセージキューで行ってください。
