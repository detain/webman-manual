# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) は [top-think/think-orm](https://github.com/top-think/think-orm) を基に開発されたデータベースコンポーネントで、コネクションプール、コルーチン環境および非コルーチン環境の両方をサポートしています。

## think-ormのインストール

`composer require -W webman/think-orm`

インストール後はrestart（再起動）が必要です（reloadは無効です）。

## 設定ファイル

実際の状況に応じて設定ファイル `config/think-orm.php` を修正してください。

## ドキュメント

https://www.kancloud.cn/manual/think-orm

## 使用方法

```php
<?php
namespace app\controller;

use support\Request;
use support\think\Db;

class FooController
{
    public function get(Request $request)
    {
        $user = Db::table('user')->where('uid', '>', 1)->find();
        return json($user);
    }
}
```

## モデルの作成

think-ormモデルは `support\think\Model` を継承します。以下を参考にしてください。

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $pk = 'id';

}
```

以下のコマンドでも、think-ormベースのモデルを作成できます。

```
php webman make:model テーブル名
```

> **ヒント**
> このコマンドには `webman/console` のインストールが必要です。インストールコマンド：`composer require webman/console ^1.2.13`

> **注意**
> make:model コマンドは、親プロジェクトで `illuminate/database` を使用していることを検出すると、think-orm ではなく `illuminate/database` ベースのモデルファイルを作成します。その場合は、追加パラメータ `tp` を付けて think-orm のモデルを強制的に生成できます。例：`php webman make:model テーブル名 tp`（動作しない場合は `webman/console` をアップグレードしてください）。
