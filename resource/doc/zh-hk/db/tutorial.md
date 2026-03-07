# 資料庫快速入門（基於 Laravel 資料庫組件）

[webman/database](https://github.com/webman-php/database) 基於 [illuminate/database](https://github.com/illuminate/database) 開發，並加入連線池功能，支援協程與非協程環境，用法與 Laravel 相同。

開發者也可以參考 [使用其他資料庫組件](others.md) 章節，使用 ThinkPHP 或其他資料庫。

## 資料庫安裝

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

安裝後需使用 restart 重新啟動（reload 無效）。

> **提示**
> webman/database 依賴 Laravel 的 `illuminate/database`，安裝時會自動安裝其依賴套件。

> **注意**
> 若不需要分頁、資料庫事件、記錄 SQL，只需要執行：
> `composer require -W webman/database`

## 資料庫設定
`config/database.php`
```php

return [
    // 預設資料庫
    'default' => 'mysql',

    // 各類資料庫連線設定
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
                PDO::ATTR_EMULATE_PREPARES => false, // 使用 swoole 或 swow 作為執行環境時為必填
            ],
            'pool' => [ // 連線池設定
                'max_connections' => 5, // 最大連線數
                'min_connections' => 1, // 最小連線數
                'wait_timeout' => 3,    // 從連線池取得連線的最長等待時間，逾時會拋出異常。僅在協程環境有效
                'idle_timeout' => 60,   // 連線池中連線的最大閒置時間，逾時後會關閉回收，直到連線數為 min_connections
                'heartbeat_interval' => 50, // 連線池心跳檢測間隔，單位為秒，建議小於 60 秒
            ],
        ],
    ],
];
```

除 `pool` 設定外，其餘與 Laravel 相同。

## 關於連線池
* 每個行程有獨立的連線池，行程間不共用連線池。
* 未啟用協程時，請求在行程內依序執行，無並發，因此連線池最多只有 1 個連線。
* 啟用協程後，請求在行程內並發執行，連線池會依需求動態調整連線數，最多不超過 `max_connections`，最少不低於 `min_connections`。
* 因為連線池連線數上限為 `max_connections`，當操作資料庫的協程數超過此值時，多餘協程會排隊等待，最多等待 `wait_timeout` 秒，逾時會觸發異常。
* 在閒置情況下（含協程與非協程環境），連線會在 `idle_timeout` 後被回收，直到連線數為 `min_connections`（`min_connections` 可為 0）。


## 資料庫使用範例
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

用法與 Laravel 相同，使用 `Db::table()` 方法操作資料庫。
