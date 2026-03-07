# Redis 佇列

基於 Redis 的訊息佇列，支援訊息延遲處理。

## 安裝
`composer require webman/redis-queue`

## 設定檔
Redis 設定檔會自動產生於 `{主專案}/config/plugin/webman/redis-queue/redis.php`，內容類似如下：
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // 密碼，可選參數
            'db' => 0,            // 資料庫
            'max_attempts'  => 5, // 消費失敗後，重試次數
            'retry_seconds' => 5, // 重試間隔，單位秒
        ]
    ],
];
```

### 消費失敗重試
如果消費失敗（發生異常），則訊息會放入延遲佇列，等待下次重試。重試次數由參數 `max_attempts` 控制，重試間隔由 `retry_seconds` 和 `max_attempts` 共同控制。例如 `max_attempts` 為 5，`retry_seconds` 為 10，第 1 次重試間隔為 `1*10` 秒，第 2 次為 `2*10` 秒，第 3 次為 `3*10` 秒，以此類推直到重試 5 次。若超過 `max_attempts` 設定的重試次數，則訊息放入 key 為 `{redis-queue}-failed` 的失敗佇列。

## 投遞訊息（同步）

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // 佇列名稱
        $queue = 'send-mail';
        // 資料，可以直接傳陣列，無需序列化
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // 投遞訊息
        Redis::send($queue, $data);
        // 投遞延遲訊息，訊息會在 60 秒後處理
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
投遞成功時 `Redis::send()` 回傳 true，否則回傳 false 或拋出例外。

> **提示**
> 延遲佇列消費時間可能會有誤差，例如消費速度小於生產速度導致佇列積壓，進而導致消費延遲，緩解方法是多開一些消費行程。

## 投遞訊息（非同步）
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // 佇列名稱
        $queue = 'send-mail';
        // 資料，可以直接傳陣列，無需序列化
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // 投遞訊息
        Client::send($queue, $data);
        // 投遞延遲訊息，訊息會在 60 秒後處理
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` 沒有回傳值，屬於非同步推送，不保證訊息 100% 送達 Redis。

> **提示**
> `Client::send()` 的原理是在本機記憶體建立一個記憶體佇列，非同步將訊息同步到 Redis（同步速度很快，每秒約 1 萬筆訊息）。若行程重啟時本機記憶體佇列內的資料尚未同步完畢，會造成訊息遺失。`Client::send()` 非同步投遞適合投遞不重要的訊息。

> **提示**
> `Client::send()` 為非同步，僅能在 workerman 的執行環境中使用，命令列腳本請使用同步介面 `Redis::send()`。

## 在其他專案投遞訊息
有時候需要在其他專案中投遞訊息且無法使用 `webman\redis-queue`，則可參考以下函式向佇列投遞訊息。

```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting';
    $queue_delay = '{redis-queue}-delayed';
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```

其中，參數 `$redis` 為 Redis 實例。例如 redis 擴充套件用法類似如下：
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## 消費
消費行程設定檔位於 `{主專案}/config/plugin/webman/redis-queue/process.php`。 consumer 目錄位於 `{主專案}/app/queue/redis/` 下。

執行命令 `php webman redis-queue:consumer my-send-mail` 會產生檔案 `{主專案}/app/queue/redis/MyMailSend.php`

> **提示**
> 以上命令需安裝 [命令列](../plugin/console.md)，若不欲安裝也可手動產生類似如下：

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // 要消費的佇列名稱
    public $queue = 'send-mail';

    // 連線名稱，對應 plugin/webman/redis-queue/redis.php 裡的連線
    public $connection = 'default';

    // 消費
    public function consume($data)
    {
        // 無需反序列化
        var_export($data); // 輸出 ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // 消費失敗回呼
    /* 
    $package = [
        'id' => 1357277951, // 訊息 ID
        'time' => 1709170510, // 訊息時間
        'delay' => 0, // 延遲時間
        'attempts' => 2, // 消費次數
        'queue' => 'send-mail', // 佇列名稱
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // 訊息內容
        'max_attempts' => 5, // 最大重試次數
        'error' => '錯誤訊息' // 錯誤訊息
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // 無需反序列化
        var_export($package); 
    }
}
```

> **注意**
> 消費過程中沒有拋出例外和 Error 視為消費成功，否則視為消費失敗，進入重試佇列。redis-queue 沒有 ack 機制，可視為自動 ack（未產生例外或 Error）。若消費過程中欲標記當前訊息消費不成功，可手動拋出例外，使當前訊息進入重試佇列，實質上與 ack 機制無異。

> **提示**
> consumer 支援多伺服器多行程，且同一則訊息**不會**被重複消費。已消費的訊息會自動自佇列刪除，無需手動刪除。

> **提示**
> 消費行程可同時消費多種不同的佇列，新增佇列不需修改 `process.php` 中的設定，新增佇列 consumer 時只需在 `app/queue/redis` 下新增對應的 `Consumer` 類別，並以類別屬性 `$queue` 指定要消費的佇列名稱。

> **提示**
> Windows 使用者需執行 `php windows.php` 啟動 webman，否則消費行程不會啟動。

> **提示**
> onConsumeFailure 回呼會在每次消費失敗時觸發，可在此處理失敗後的邏輯。（此功能需 `webman/redis-queue>=1.3.2`、`workerman/redis-queue>=1.2.1`）

## 為不同佇列設定不同消費行程
預設情況下，所有 consumer 共用相同的消費行程。但有時需將部分佇列的消費獨立出來，例如將消費慢的業務放到一組行程中消費，消費快的業務放到另一組行程消費。可將 consumer 分為兩個目錄，例如 `app_path() . '/queue/redis/fast'` 和 `app_path() . '/queue/redis/slow'`（注意消費類別的命名空間需相應變更），設定如下：
```php
return [
    ...此處省略其他設定...
    
    'redis_consumer_fast'  => [ // key 為自訂，無格式限制，此處命名為 redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // consumer 類別目錄
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // key 為自訂，無格式限制，此處命名為 redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // consumer 類別目錄
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

如此快業務 consumer 放在 `queue/redis/fast` 目錄下，慢業務 consumer 放在 `queue/redis/slow` 目錄下，可達到為佇列指定消費行程的目的。

## 多 Redis 設定
#### 設定
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // 密碼，字串型別，可選參數
            'db' => 0,            // 資料庫
            'max_attempts'  => 5, // 消費失敗後，重試次數
            'retry_seconds' => 5, // 重試間隔，單位秒
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // 密碼，字串型別，可選參數
            'db' => 0,            // 資料庫
            'max_attempts'  => 5, // 消費失敗後，重試次數
            'retry_seconds' => 5, // 重試間隔，單位秒
        ]
    ],
];
```

注意設定中已新增以 `other` 為 key 的 Redis 設定。

#### 多 Redis 投遞訊息

```php
// 向 key 為 `default` 的佇列投遞訊息
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// 等同於
Client::send($queue, $data);
Redis::send($queue, $data);

// 向 key 為 `other` 的佇列投遞訊息
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### 多 Redis 消費
消費設定中 key 為 `other` 的佇列：
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // 要消費的佇列名稱
    public $queue = 'send-mail';

    // === 此處設為 other，代表消費設定中 key 為 other 的佇列 ===
    public $connection = 'other';

    // 消費
    public function consume($data)
    {
        // 無需反序列化
        var_export($data);
    }
}
```

## 常見問題

**為何會出現錯誤 `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)`**

此錯誤僅會出現於非同步投遞介面 `Client::send()`。非同步投遞會先將訊息儲存於本機記憶體，於行程閒置時再將訊息傳送給 Redis。若 Redis 接收速度慢於訊息生產速度，或行程持續忙碌於其他業務，未有足夠時間將記憶體中的訊息同步至 Redis，即會導致訊息積壓。若有訊息積壓超過 600 秒，即會觸發此錯誤。

解決方案：投遞訊息請使用同步投遞介面 `Redis::send()`。
