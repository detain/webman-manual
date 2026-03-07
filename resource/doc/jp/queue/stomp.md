# Stompキュー

Stompはシンプル（ストリーム）テキスト指向メッセージプロトコルで、相互運用可能な接続形式を提供し、STOMPクライアントが任意のSTOMPメッセージブローカー（Broker）とやり取りできるようにします。[workerman/stomp](https://github.com/walkor/stomp) はStompクライアントを実装し、主にRabbitMQ、Apollo、ActiveMQなどのメッセージキューシーンで使用されます。

## インストール
`composer require webman/stomp`

## 設定
設定ファイルは `config/plugin/webman/stomp` にあります。

## メッセージの送信
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // キュー名
        $queue = 'examples';
        // データ（配列を渡す場合は、json_encode、serializeなどを使用して独自にシリアライズする必要があります）
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // 送信を実行
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> 他プロジェクトとの互換性のため、Stompコンポーネントは自動シリアライズ・デシリアライズ機能を提供していません。配列データを送信する場合は独自にシリアライズし、消費時に独自にデシリアライズする必要があります。

## メッセージの消費
`app/queue/stomp/MyMailSend.php` を新規作成します（クラス名は任意で、PSR-4規約に準拠すれば可）。
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // キュー名
    public $queue = 'examples';

    // 接続名、stomp.php内の接続に対応
    public $connection = 'default';

    // 値が client の場合は、$ack_resolver->ack() を呼び出してサーバーに消費成功を通知する必要があります
    // 値が auto の場合は、$ack_resolver->ack() を呼び出す必要はありません
    public $ack = 'auto';

    // 消費
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // データが配列の場合は、独自にデシリアライズする必要があります
        var_export(json_decode($data, true)); // ['to' => 'tom@gmail.com', 'content' => 'hello'] を出力
        // サーバーに消費成功を通知
        $ack_resolver->ack(); // ack が auto の場合はこの呼び出しを省略できます
    }
}
```

# RabbitMQでStompプロトコルを有効にする
RabbitMQはデフォルトでStompプロトコルが有効になっていません。以下のコマンドを実行して有効化する必要があります。
```
rabbitmq-plugins enable rabbitmq_stomp
```
有効化後、Stompのデフォルトポートは61613です。
