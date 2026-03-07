# Redisキュー

Redisベースのメッセージキュー。メッセージの遅延処理をサポートします。

## インストール
`composer require webman/redis-queue`

## 設定ファイル
Redis設定ファイルは `{メインプロジェクト}/config/plugin/webman/redis-queue/redis.php` に自動生成され、内容は以下のようになります：
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // パスワード、オプション
            'db' => 0,            // データベース
            'max_attempts'  => 5, // 消費失敗時のリトライ回数
            'retry_seconds' => 5, // リトライ間隔（秒）
        ]
    ],
];
```

### 消費失敗時のリトライ
消費が失敗した場合（例外が発生した場合）、メッセージは遅延キューに入り、次回のリトライを待ちます。リトライ回数は `max_attempts` で、リトライ間隔は `retry_seconds` と `max_attempts` の組み合わせで制御されます。例えば `max_attempts` が5、`retry_seconds` が10の場合、1回目は `1*10` 秒後、2回目は `2*10` 秒後、3回目は `3*10` 秒後…と5回までリトライされます。`max_attempts` のリトライ回数を超えると、メッセージは `{redis-queue}-failed` キーの失敗キューに入ります。

## メッセージ送信（同期）

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // キュー名
        $queue = 'send-mail';
        // データ、配列を直接渡せます、シリアライズ不要
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // メッセージ送信
        Redis::send($queue, $data);
        // 遅延メッセージ送信、60秒後に処理
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
送信成功時 `Redis::send()` は true を、それ以外は false または例外を返します。

> **ヒント**
> 遅延キューの消費時刻にずれが生じる場合があります。例えば消費速度が生産速度より遅いとキューが滞留し消費が遅れます。対策：消費プロセスを増やしてください。

## メッセージ送信（非同期）
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // キュー名
        $queue = 'send-mail';
        // データ、配列を直接渡せます、シリアライズ不要
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // メッセージ送信
        Client::send($queue, $data);
        // 遅延メッセージ送信、60秒後に処理
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` は戻り値がありません。非同期プッシュであり、Redis への100%送達を保証しません。

> **ヒント**
> `Client::send()` の仕組みは、ローカルメモリにメモリキューを作り、メッセージを非同期で Redis に同期することです（同期は高速、秒間約1万メッセージ）。プロセス再起動時にメモリキュー内のデータが同期しきれていないと、メッセージが失われる可能性があります。`Client::send()` の非同期送信は重要なメッセージ以外に向いています。

> **ヒント**
> `Client::send()` は非同期のため、Workerman の実行環境でのみ使用できます。コマンドラインスクリプトでは同期インターフェース `Redis::send()` を使ってください。

## 他プロジェクトからのメッセージ送信
他プロジェクトからメッセージを送る必要があり `webman\redis-queue` が使えない場合、以下の関数を参考にキューへメッセージを送れます。

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

ここで `$redis` は Redis インスタンスです。redis 拡張の使用例：

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## 消費
消費プロセス設定ファイルは `{メインプロジェクト}/config/plugin/webman/redis-queue/process.php` にあります。 consumer ディレクトリは `{メインプロジェクト}/app/queue/redis/` の下です。

コマンド `php webman redis-queue:consumer my-send-mail` を実行すると `{メインプロジェクト}/app/queue/redis/MyMailSend.php` が生成されます。

> **ヒント**
> このコマンドには [コンソール](../plugin/console.md) プラグインのインストールが必要です。インストールしない場合は、以下のようなファイルを手動で作成できます：

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // 消費するキュー名
    public $queue = 'send-mail';

    // 接続名、plugin/webman/redis-queue/redis.php の接続に対応
    public $connection = 'default';

    // 消費
    public function consume($data)
    {
        // デシリアライズ不要
        var_export($data); // 出力 ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // 消費失敗コールバック
    /* 
    $package = [
        'id' => 1357277951, // メッセージID
        'time' => 1709170510, // メッセージ時刻
        'delay' => 0, // 遅延時間
        'attempts' => 2, // 消費回数
        'queue' => 'send-mail', // キュー名
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // メッセージ内容
        'max_attempts' => 5, // 最大リトライ回数
        'error' => 'エラーメッセージ' // エラーメッセージ
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // デシリアライズ不要
        var_export($package); 
    }
}
```

> **注意**
> 消費中に例外や Error を投げなければ消費成功、そうでなければ失敗でリトライキューに入ります。redis-queue には ack 機構がなく、自動 ack と考えてよいです（例外・Error が出なければ）。現在のメッセージを消費失敗としてマークしたい場合は、手動で例外を投げてリトライキューに送れます。実質 ack 機構と同じです。

> **ヒント**
> consumer は複数サーバー・プロセスに対応し、同一メッセージは二重に消費されません。消費済みメッセージは自動でキューから削除されます。

> **ヒント**
> 消費プロセスは複数の異なるキューを同時に消費できます。新規キュー追加時は `process.php` の設定変更は不要です。新規キュー consumer 追加時は `app/queue/redis` の下に対応する `Consumer` クラスを追加し、クラスプロパティ `$queue` で消費するキュー名を指定してください。

> **ヒント**
> Windows ユーザーは webman 起動に `php windows.php` を実行する必要があります。実行しないと消費プロセスは起動しません。

> **ヒント**
> onConsumeFailure コールバックは消費失敗のたびに呼ばれます。ここで失敗後の処理を書けます。（この機能には `webman/redis-queue>=1.3.2` と `workerman/redis-queue>=1.2.1` が必要です）

## キューごとに異なる消費プロセスを設定
デフォルトではすべての consumer が同じ消費プロセスを共有します。一部のキューの消費を分けたい場合（例：遅い処理を一つのプロセス群、速い処理を別のプロセス群で処理）は、consumer を二つのディレクトリに分けられます。例：`app_path() . '/queue/redis/fast'` と `app_path() . '/queue/redis/slow'`（consumer クラスの名前空間も合わせて変更が必要）。設定例：
```php
return [
    ...他の設定は省略...
    
    'redis_consumer_fast'  => [ // キーは任意、形式制限なし、ここでは redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // consumer クラスのディレクトリ
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // キーは任意、形式制限なし、ここでは redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // consumer クラスのディレクトリ
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

これで高速処理の consumer は `queue/redis/fast`、低速処理の consumer は `queue/redis/slow` に入り、キューごとに消費プロセスを割り当てられます。

## 複数 Redis 設定
#### 設定
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // パスワード、文字列、オプション
            'db' => 0,           // データベース
            'max_attempts'  => 5, // 消費失敗時のリトライ回数
            'retry_seconds' => 5, // リトライ間隔（秒）
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // パスワード、文字列、オプション
            'db' => 0,            // データベース
            'max_attempts'  => 5, // 消費失敗時のリトライ回数
            'retry_seconds' => 5, // リトライ間隔（秒）
        ]
    ],
];
```

設定にキー `other` の Redis 設定が追加されています。

#### 複数 Redis へのメッセージ送信

```php
// キー `default` のキューへメッセージ送信
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// 以下と同じ
Client::send($queue, $data);
Redis::send($queue, $data);

// キー `other` のキューへメッセージ送信
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### 複数 Redis からの消費
設定でキー `other` のキューからメッセージを消費：
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // 消費するキュー名
    public $queue = 'send-mail';

    // === 設定のキー 'other' のキューを消費するにはここを 'other' に設定 ===
    public $connection = 'other';

    // 消費
    public function consume($data)
    {
        // デシリアライズ不要
        var_export($data);
    }
}
```

## よくある質問

**エラー `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)` が発生する理由**

このエラーは非同期送信インターフェース `Client::send()` でのみ発生します。非同期送信はメッセージをまずローカルメモリに保存し、プロセスがアイドルになったら Redis に送ります。Redis の受信速度がメッセージ生成速度より遅い、あるいはプロセスが他処理で忙しくメモリのメッセージを Redis に同期する時間が足りないと、メッセージが滞留します。600秒以上滞留するとこのエラーが発生します。

対処：メッセージ送信には同期送信インターフェース `Redis::send()` を使ってください。
