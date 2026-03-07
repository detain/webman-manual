# 資料庫

由於大部分插件都會安裝[webman-admin](https://www.workerman.net/plugin/82)，所以建議直接複用`webman-admin`的資料庫設定。

模型基底類別使用`plugin\admin\app\model\Base`則會自動使用`webman-admin`的資料庫設定。
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

也可以透過`plugin.admin.mysql`操作`webman-admin`的資料庫，例如

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## 使用自己的資料庫
插件也可以選擇使用自己的資料庫，例如`plugin/foo/config/database.php`內容如下
```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql 為連線名稱
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => '資料庫',
            'username'    => '使用者名稱',
            'password'    => '密碼',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin 為連線名稱
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => '資料庫',
            'username'    => '使用者名稱',
            'password'    => '密碼',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```
引用方式為`Db::connection('plugin.{插件}.{連線名稱}');`，例如
```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

如果想使用主專案的資料庫，則直接使用即可，例如
```php
use support\Db;
Db::table('user')->first();
// 假設主專案還設定了一個 admin 連線
Db::connection('admin')->table('admin')->first();
```

#### 為 Model 設定資料庫

我們可以為 Model 建立一個 Base 類別，Base 類別用`$connection`指定插件自己的資料庫連線，例如

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

這樣插件裡所有的 Model 繼承自 Base，就自動使用了插件自己的資料庫。

## 自動匯入資料庫
執行 `php webman app-plugin:create foo` 會自動建立 foo 插件，其中包含 `plugin/foo/api/Install.php` 和 `plugin/foo/install.sql`。

> **提示**
> 如果沒有產生 install.sql 檔案，請嘗試升級`webman/console`，指令為`composer require webman/console ^1.3.6`

#### 插件安裝時匯入資料庫
安裝插件時會執行 Install.php 裡的`install`方法，該方法會自動匯入`install.sql`裡的 SQL 語句，從而實現自動匯入資料庫資料表的目的。

`install.sql`檔案內容是建立資料庫資料表以及對資料表歷史修改的 SQL 語句，注意每個語句必須用`;`結束，例如
```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '主鍵',
  `order_id` varchar(50) NOT NULL COMMENT '訂單id',
  `user_id` int NOT NULL COMMENT '使用者id',
  `total_amount` decimal(10,2) NOT NULL COMMENT '須支付金額',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='訂單';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '主鍵',
  `name` varchar(50) NOT NULL COMMENT '名稱',
  `price` int NOT NULL COMMENT '價格',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='商品';
```

**變更資料庫連線**

`install.sql`匯入的目標資料庫預設為`webman-admin`的資料庫，如果想匯入到其它資料庫，可以修改`Install.php`裡的`$connection`屬性，例如
```php
<?php

class Install
{
    // 指定插件自己的資料庫
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**測試**

執行`php webman app-plugin:install foo`安裝插件，然後查看資料庫，會發現`foo_orders`和`foo_goods`資料表已經建立了。

#### 插件升級時變更資料表結構
有時候插件升級需要新建資料表或變更資料表結構，可以直接在`install.sql`後面追加對應的語句即可，注意每個語句後面用`;`結束，例如追加一個`foo_user`資料表以及給`foo_orders`資料表增加一個`status`欄位
```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '主鍵',
  `order_id` varchar(50) NOT NULL COMMENT '訂單id',
  `user_id` int NOT NULL COMMENT '使用者id',
  `total_amount` decimal(10,2) NOT NULL COMMENT '須支付金額',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='訂單';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT '主鍵',
 `name` varchar(50) NOT NULL COMMENT '名稱',
 `price` int NOT NULL COMMENT '價格',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='商品';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT '主鍵',
 `name` varchar(50) NOT NULL COMMENT '名字'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='使用者';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT '狀態';
```

升級時會執行 Install.php 裡的`update`方法，該方法同樣會執行`install.sql`裡的語句，如果有新的語句會執行新的語句，如果是舊的語句會略過，從而實現升級對資料庫的修改。

#### 解除安裝插件時刪除資料庫
解除安裝插件時`Install.php`的`uninstall`方法會被呼叫，它會自動分析`install.sql`裡有哪些建表語句，並自動刪除這些資料表，達到解除安裝插件時刪除資料庫資料表的目的。
如果想解除安裝時只想執行自己的`uninstall.sql`，不執行自動的刪表操作，則只需要建立`plugin/插件名/uninstall.sql`即可，這樣`uninstall`方法只會執行`uninstall.sql`檔案裡的語句。
