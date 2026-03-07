# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) 是基於 [top-think/think-orm](https://github.com/top-think/think-orm) 開發的資料庫元件，支援連線池，支援協程和非協程環境。

## 安裝 think-orm

`composer require -W webman/think-orm`

安裝後需要 restart 重新啟動（reload 無效）。

## 設定檔

根據實際情況修改設定檔 `config/think-orm.php`。

## 文件網址

https://www.kancloud.cn/manual/think-orm

## 使用

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

## 建立模型

think-orm 模型繼承 `support\think\Model`，類似如下：

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

你也可以使用以下指令建立基於 think-orm 的模型：

```
php webman make:model 表名
```

> **提示**
> 此指令需要安裝 `webman/console`，安裝指令為 `composer require webman/console ^1.2.13`。

> **注意**
> make:model 指令如果偵測到主專案使用了 `illuminate/database`，會建立基於 `illuminate/database` 的模型檔案，而不是 think-orm 的。這時可以透過附加參數 `tp` 來強制產生 think-orm 的模型，指令類似 `php webman make:model 表名 tp`（如果不生效請升級 `webman/console`）。
