# Medoo 資料庫

[webman/medoo](https://github.com/webman-php/medoo) 在 [Medoo](https://medoo.in/) 的基礎上增加了連接池功能，並支援協程和非協程環境，用法與 Medoo 相同。

## 安裝
`composer require webman/medoo`

## Medoo 資料庫配置
配置檔案位置：`config/plugin/webman/medoo/database.php`

## Medoo 資料庫使用
```php
<?php
namespace app\controller;

use support\Request;
use support\Medoo;

class Index
{
    public function index(Request $request)
    {
        $user = Medoo::get('user', '*', ['uid' => 1]);
        return json($user);
    }
}
```

> **提示**
> `Medoo::get('user', '*', ['uid' => 1]);`
> 等同於
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## Medoo 多資料庫配置

**配置**
在 `config/plugin/webman/medoo/database.php` 裡新增一個配置，key 任意，這裡使用的是 `other`。

```php
<?php
return [
    'default' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [ // 連接池配置
            'max_connections' => 5, // 最大連接數
            'min_connections' => 1, // 最小連接數
            'wait_timeout' => 60,   // 從連接池獲取連接等待的最大時間，超時後會拋出異常
            'idle_timeout' => 3,    // 連接池中連接最大空閒時間，超時後會關閉回收，直到連接數為 min_connections
            'heartbeat_interval' => 50, // 連接池心跳檢測時間，單位秒，建議小於 60 秒
        ]
    ],
    // 這裡新增了一個 other 的配置
    'other' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [
            'max_connections' => 5,
            'min_connections' => 1,
            'wait_timeout' => 60,
            'idle_timeout' => 3,
            'heartbeat_interval' => 50,
        ],
    ],
];
```

## Medoo 資料庫使用
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

參見 [Medoo 官方文檔](https://medoo.in/api/select)
