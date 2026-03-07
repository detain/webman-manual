# データベース

ほとんどのプラグインが[webman-admin](https://www.workerman.net/plugin/82)をインストールするため、`webman-admin`のデータベース設定をそのまま再利用することを推奨します。

モデルの基底クラスに`plugin\admin\app\model\Base`を使用すると、自動的にwebman-adminのデータベース設定が適用されます。
```php
<?php

namespace plugin\foo\app\model;

use plugin\admin\app\model\Base;

class Orders extends Base
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'foo_orders';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $primaryKey = 'id';
    
}
```

`plugin.admin.mysql`経由でwebman-adminのデータベースにアクセスすることもできます。例：

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## 独自のデータベースを使用する

プラグインは独自のデータベースを使用することも選択できます。例えば`plugin/foo/config/database.php`の内容：

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysqlは接続名
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'データベース',
            'username'    => 'ユーザー名',
            'password'    => 'パスワード',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // adminは接続名
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'データベース',
            'username'    => 'ユーザー名',
            'password'    => 'パスワード',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```
参照形式は`Db::connection('plugin.{プラグイン}.{接続名}');`です。例：
```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

メインプロジェクトのデータベースを使う場合は、直接呼び出してください。例：
```php
use support\Db;
Db::table('user')->first();
// メインプロジェクトにもadmin接続が設定されている場合
Db::connection('admin')->table('admin')->first();
```

#### Model用のデータベース設定

Model用のBaseクラスを作成し、`$connection`でプラグイン独自のデータベース接続を指定できます。例：

```php
<?php

namespace plugin\foo\app\model;

use DateTimeInterface;
use support\Model;

class Base extends Model
{
    /**
     * @var string
     */
    protected $connection = 'plugin.foo.mysql';

}
```

これにより、プラグイン内の全てのModelがBaseを継承するだけで、プラグイン独自のデータベースを自動的に使用します。

## データベースの自動インポート
`php webman app-plugin:create foo`を実行すると、fooプラグインが自動作成され、`plugin/foo/api/Install.php`と`plugin/foo/install.sql`が含まれます。

> **ヒント**
> install.sqlファイルが生成されない場合は、`webman/console`のアップグレードを試してください：`composer require webman/console ^1.3.6`

#### プラグインインストール時のデータベースインポート
プラグインのインストール時に、Install.phpの`install`メソッドが実行され、`install.sql`内のSQL文が自動実行されて、データベーステーブルがインポートされます。

`install.sql`の内容は、テーブル作成およびテーブルの履歴変更SQL文であり、各文は`;`で終了する必要があります。例：
```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '主キー',
  `order_id` varchar(50) NOT NULL COMMENT '注文ID',
  `user_id` int NOT NULL COMMENT 'ユーザーID',
  `total_amount` decimal(10,2) NOT NULL COMMENT '支払金額',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='注文';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '主キー',
  `name` varchar(50) NOT NULL COMMENT '名称',
  `price` int NOT NULL COMMENT '価格',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='商品';
```

**データベース接続の変更**

`install.sql`のインポート先はデフォルトでwebman-adminのデータベースです。他のデータベースにインポートするには、`Install.php`の`$connection`プロパティを変更してください。例：
```php
<?php

class Install
{
    // プラグイン独自のデータベースを指定
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**テスト**

`php webman app-plugin:install foo`を実行してプラグインをインストールし、データベースを確認すると、`foo_orders`と`foo_goods`テーブルが作成されているはずです。

#### プラグインアップグレード時のテーブル構造変更
プラグインのアップグレードで新規テーブルの作成やテーブル構造の変更が必要な場合、`install.sql`の末尾に対応する文を追加してください。各文は`;`で終了する必要があります。例：`foo_user`テーブルと`foo_orders`テーブルへの`status`カラム追加
```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '主キー',
  `order_id` varchar(50) NOT NULL COMMENT '注文ID',
  `user_id` int NOT NULL COMMENT 'ユーザーID',
  `total_amount` decimal(10,2) NOT NULL COMMENT '支払金額',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='注文';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT '主キー',
 `name` varchar(50) NOT NULL COMMENT '名称',
 `price` int NOT NULL COMMENT '価格',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='商品';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT '主キー',
 `name` varchar(50) NOT NULL COMMENT '名前'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='ユーザー';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT '状態';
```

アップグレード時にはInstall.phpの`update`メソッドが実行され、同様に`install.sql`内の文を実行します。新しい文があれば実行され、既に実行済みの文はスキップされるため、アップグレード時のデータベース変更が正しく適用されます。

#### プラグインアンインストール時のデータベース削除
プラグインのアンインストール時に、Install.phpの`uninstall`メソッドが呼び出されます。このメソッドは`install.sql`内のCREATE TABLE文を自動解析し、それらのテーブルを削除して、アンインストール時にデータベーステーブルを削除します。
アンインストール時に独自の`uninstall.sql`のみを実行し、自動的なテーブル削除を行いたくない場合は、`plugin/プラグイン名/uninstall.sql`を作成してください。これにより`uninstall`メソッドは`uninstall.sql`ファイル内の文のみを実行します。
