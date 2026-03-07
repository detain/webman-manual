# Stomp隊列

Stomp是簡單(流)文字定向訊息協定，它提供了一個可互操作的連接格式，允許STOMP客戶端與任意STOMP訊息代理（Broker）進行交互。[workerman/stomp](https://github.com/walkor/stomp)實現了Stomp客戶端，主要用於 RabbitMQ、Apollo、ActiveMQ 等訊息佇列場景。

## 安裝
`composer require webman/stomp`

## 配置
配置檔案在 `config/plugin/webman/stomp` 下

## 投遞訊息
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // 佇列
        $queue = 'examples';
        // 資料（傳遞陣列時需要自行序列化，例如使用json_encode、serialize等）
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // 執行投遞
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> 為了相容其他專案，Stomp元件沒有提供自動序列化與反序列化功能。若投遞的是陣列資料，需要自行序列化，消費時自行反序列化。

## 消費訊息
新建 `app/queue/stomp/MyMailSend.php`（類別名稱可自訂，符合 PSR-4 規範即可）。
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // 佇列名稱
    public $queue = 'examples';

    // 連接名稱，對應 stomp.php 中的連接
    public $connection = 'default';

    // 值為 client 時需要調用 $ack_resolver->ack() 告知服務端已成功消費
    // 值為 auto 時無需調用 $ack_resolver->ack()
    public $ack = 'auto';

    // 消費
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // 若資料為陣列，需自行反序列化
        var_export(json_decode($data, true)); // 輸出 ['to' => 'tom@gmail.com', 'content' => 'hello']
        // 告知服務端已成功消費
        $ack_resolver->ack(); // ack 為 auto 時可省略此呼叫
    }
}
```

# RabbitMQ 開啟 Stomp 協定
RabbitMQ 預設未開啟 Stomp 協定，需執行以下指令開啟：
```
rabbitmq-plugins enable rabbitmq_stomp
```
開啟後，Stomp 的預設埠為 61613。
