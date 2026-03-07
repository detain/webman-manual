# 正確使用事務
webman 使用資料庫事務與其它框架相同，此處列舉需要注意的事項

## 程式碼結構

程式碼結構與其它框架相同，例如 Laravel 用法（think-orm 類似）：

```php
Db::beginTransaction();
try {
    // ..業務處理略...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

特別需要注意的是**必須使用** `\Throwable` 而**不能使用** `\Exception`，因為業務處理過程中可能觸發 `Error`，它並不屬於 `Exception`

## 資料庫連線

在事務中操作模型時，特別需要注意模型是否設定了連線。若模型設定了連線，開啟事務時必須指定連線，否則事務無效（think-orm 類似）。例如：

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // 此處為模型指定了連線
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

當模型指定了連線時，開啟事務、提交事務、回滾事務都必須指定連線：

```php
Db::connection('mysql')->beginTransaction();
try {
    // 業務處理
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## 查找未提交事務的請求
有時業務程式碼的 bug 會導致某個請求的事務未提交。為快速定位是哪個控制器方法未提交事務，可安裝 `webman/log` 元件，該元件會在請求完成後自動檢查是否有未提交的事務並記錄日誌，日誌關鍵字為 `Uncommitted transactions`

**webman/log 安裝方法**

`composer require webman/log`

> **注意**
> 安裝後需使用 restart 重新啟動，reload 不會生效
