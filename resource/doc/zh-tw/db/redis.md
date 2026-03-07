# Redis

[webman/redis](https://github.com/webman-php/redis) 基於 [illuminate/redis](https://github.com/illuminate/redis) 添加了連接池功能，支援協程與非協程環境，用法與 Laravel 相同。

使用 `illuminate/redis` 之前必須先為 `php-cli` 安裝 redis 擴展。

## 安裝

```php
composer require -W webman/redis illuminate/events
```

安裝後需 restart 重啟（reload 無效）

## 配置

Redis 配置檔位於 `config/redis.php`

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
            'wait_timeout' => 3,        // 從連接池獲取連接的最大等待時間
            'idle_timeout' => 50,       // 連接空閒超時時間，超過後會關閉直到連接數為 min_connections
            'heartbeat_interval' => 50, // 心跳檢測間隔，不要大於 60 秒
        ],
    ]
];
```

## 關於連接池

* 每個進程有各自的連接池，進程間不共享。
* 未開啟協程時，業務在進程內順序執行，連接池最多只有 1 個連接。
* 開啟協程後，業務並發執行，連接池會動態調整連接數，最多不超過 `max_connections`，最少不少於 `min_connections`。
* 當操作 Redis 的協程數大於 `max_connections` 時，多餘的協程會排隊等待，最多等 `wait_timeout` 秒，超過則拋出異常。
* 空閒情況下（含協程與非協程），連接會在 `idle_timeout` 後釋放，直到連接數達到 `min_connections`（`min_connections` 可為 0）。

## 範例

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

等價於

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
> 慎用 `Redis::select($db)` 介面。由於 webman 為常駐記憶體框架，若某次請求使用 `Redis::select($db)` 切換資料庫，會影響後續請求。多資料庫時建議將不同 `$db` 配置為不同 Redis 連接。

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

預設使用 `default` 下的連接，可用 `Redis::connection()` 選擇要使用的 Redis 連接：

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## 集群配置

若應用使用 Redis 集群，應在 Redis 配置檔中以 `clusters` 鍵定義：

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

預設情況下，集群可在節點上做客戶端分片，以實現節點池與大量可用記憶體。須注意客戶端分片不處理失敗情形，此功能主要適用於從其他主資料庫取得的快取資料。若要使用 Redis 原生集群，需在配置檔的 `options` 中指定：

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

當需要一次操作中向伺服器發送多個命令時，建議使用管道命令。`pipeline` 方法接受 Redis 實例的閉包，可將所有命令發送給 Redis 實例，並在一次操作中完成：

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
