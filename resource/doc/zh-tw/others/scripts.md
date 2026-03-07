# 自訂腳本

有時我們需要撰寫一些臨時腳本，在這些腳本中可以像 webman 一樣呼叫任意的類或介面，完成例如資料導入、資料更新、統計等操作。在 webman 中這已經非常容易，例如：

**新建 `scripts/update.php`**（目錄不存在請自行建立）
```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';
require_once __DIR__ . '/../support/bootstrap.php';

use think\facade\Db;

$user = Db::table('user')->find(1);

var_dump($user);
```

當然，我們也可以使用 `webman/console` 自訂命令來完成這樣的操作，請參見 [命令行](../plugin/console.md)。
