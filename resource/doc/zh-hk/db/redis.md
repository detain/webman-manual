# Redis

[webman/redis](https://github.com/webman-php/redis) 基於 [illuminate/redis](https://github.com/illuminate/redis) 添加咗連接池功能，支援協程同非協程環境，用法同 Laravel 一樣。

使用 `illuminate/redis` 之前必須先為 `php-cli` 安裝 redis 擴展。

## 安裝

```php
composer require -W webman/redis illuminate/events
```

安裝後需要 restart 重啟（reload 無效）

## 配置

Redis 配置檔喺 `config/redis.php`

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // 連接池配置
            'max_connections' => 10,     // 連接池最大連接數
            'min_connections' => 1,     // 連接池最小連接數
            'wait_timeout' => 3,        // 從連接池攞連接嘅最大等待時間
            'idle_timeout' => 50,       // 連接空閒超時時間，超過後會關閉直到連接數為 min_connections
            'heartbeat_interval' => 50, // 心跳檢測間隔，唔好大過 60 秒
        ],
    ]
];
```

## 關於連接池

* 每個進程有自己嘅連接池，進程間唔共享。
* 冇開協程嘅時候，業務喺進程內順序執行，連接池最多得 1 個連接。
* 開咗協程之後，業務並發執行，連接池會動態調整連接數，最多唔超過 `max_connections`，最少唔少過 `min_connections`。
* 當操作 Redis 嘅協程數大過 `max_connections` 嘅時候，多咗嘅協程會排隊等，最多等 `wait_timeout` 秒，超過就會拋異常。
* 空閒情況下（包協程同非協程），連接會喺 `idle_timeout` 後釋放，直到連接數去到 `min_connections`（`min_connections` 可以係 0）。

## 示例

```php
<?php
namespace app\controller;

use support\Request;
use support\Redis;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Redis::set($key, rand());
        return response(Redis::get($key));
    }
}
```

## Redis 介面

```php
Redis::append($key, $value)
Redis::bitCount($key)
Redis::decr($key, $value)
Redis::decrBy($key, $value)
Redis::get($key)
Redis::getBit($key, $offset)
Redis::getRange($key, $start, $end)
Redis::getSet($key, $value)
Redis::incr($key, $value)
Redis::incrBy($key, $value)
Redis::incrByFloat($key, $value)
Redis::mGet(array $keys)
Redis::getMultiple(array $keys)
Redis::mSet($pairs)
Redis::mSetNx($pairs)
Redis::set($key, $value, $expireResolution = null, $expireTTL = null, $flag = null)
Redis::setBit($key, $offset, $value)
Redis::setEx($key, $ttl, $value)
Redis::pSetEx($key, $ttl, $value)
Redis::setNx($key, $value)
Redis::setRange($key, $offset, $value)
Redis::strLen($key)
Redis::del(...$keys)
Redis::exists(...$keys)
Redis::expire($key, $ttl)
Redis::expireAt($key, $timestamp)
Redis::select($dbIndex)
```

等同於

```php
$redis = Redis::connection('default');
$redis->append($key, $value)
$redis->bitCount($key)
$redis->decr($key, $value)
$redis->decrBy($key, $value)
$redis->get($key)
$redis->getBit($key, $offset)
...
```

> **注意**
> 慎用 `Redis::select($db)` 介面。因為 webman 係常駐記憶體框架，如果某次請求用 `Redis::select($db)` 切換資料庫，會影響後續請求。多資料庫建議將唔同 `$db` 配置成唔同 Redis 連接。

## 使用多個 Redis 連接

例如配置檔 `config/redis.php`：

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
    ],

    'cache' => [
        'host'     => '127.0.0.1',
        'password' => null,
        'port'     => 6379,
        'database' => 1,
    ],

]
```

默認用 `default` 下嘅連接，可以用 `Redis::connection()` 揀用邊個 redis 連接：

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## 集群配置

如果你嘅應用用 Redis 伺服器集群，應該喺 Redis 配置檔用 clusters 鍵定義：

```php
return [
    'clusters' => [
        'default' => [
            [
                'host'     => 'localhost',
                'username' => null,
                'password' => null,
                'port'     => 6379,
                'database' => 0,
            ],
        ],
    ],

];
```

默認情況下，集群可以喺節點上做客戶端分片，實現節點池同大量可用記憶體。要留意客戶端共享唔處理失敗情況，所以呢個功能主要適用於從另一個主資料庫攞嘅緩存資料。如果要喺 Redis 原生集群，需要喺配置檔嘅 options 鍵咁指定：

```php
return[
    'options' => [
        'cluster' => 'redis',
    ],

    'clusters' => [
        // ...
    ],
];
```

## 管道命令

當你需要喺一次操作入面向伺服器發送好多命令嘅時候，建議用管道命令。pipeline 方法接受 Redis 實例嘅閉包，你可以將所有命令發去 Redis 實例，佢哋都會喺一次操作入面完成：

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
