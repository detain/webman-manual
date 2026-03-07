# トランザクションの正しい使い方
webmanのデータベーストランザクションの使い方は他のフレームワークと同様です。ここでは注意すべき点を挙げます。

## コード構造

コード構造は他のフレームワーク（Laravel用法やthink-ormなど）と同様です：

```php
Db::beginTransaction();
try {
    // ..業務処理省略...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

特に注意すべき点として、`\Exception` ではなく **必ず** `\Throwable` を使用すること。業務処理中に `Error` が発生する可能性があり、`Error` は `Exception` を継承していないためです。

## データベース接続

トランザクション内でモデルを操作する際は、モデルに接続が設定されているかどうかに特に注意してください。モデルに接続が設定されている場合、トランザクション開始時にその接続を指定する必要があります。指定しないとトランザクションは効かないためです（think-ormも同様）。例：

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // モデルに接続を指定
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

モデルに接続が指定されている場合、トランザクションの開始・コミット・ロールバックのいずれにも接続を指定する必要があります：

```php
Db::connection('mysql')->beginTransaction();
try {
    // 業務処理
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## 未コミットのトランザクションを検出する
業務コードのバグにより、あるリクエストのトランザクションがコミットされない場合があります。未コミットのトランザクションがあるコントローラー・メソッドを素早く特定するには、`webman/log` コンポーネントをインストールしてください。このコンポーネントはリクエスト完了後に未コミットのトランザクションを自動チェックし、ログに記録します。ログのキーワードは `Uncommitted transactions` です。

**webman/log のインストール方法**

`composer require webman/log`

> **注意**
> インストール後は restart で再起動が必要です。reload では反映されません。
