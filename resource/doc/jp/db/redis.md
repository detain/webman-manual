# Redis

[webman/redis](https://github.com/webman-php/redis) は [illuminate/redis](https://github.com/illuminate/redis) を拡張し、コネクションプール機能を追加したものです。コルーチン環境と非コルーチン環境の両方をサポートし、Laravel と同じ使い方です。

`illuminate/redis` を使用する前に、`php-cli` 用の Redis 拡張をインストールする必要があります。

## インストール

```php
composer require -W webman/redis illuminate/events
```

インストール後は restart が必要です（reload では反映されません）。

## 設定

Redis の設定ファイルは `config/redis.php` にあります。

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // コネクションプール設定
            'max_connections' => 10,     // プールの最大接続数
            'min_connections' => 1,      // プールの最小接続数
            'wait_timeout' => 3,         // 接続を取得する際の最大待機時間（秒）
            'idle_timeout' => 50,        // アイドルタイムアウト。超過後は min_connections になるまで接続が閉じられる
            'heartbeat_interval' => 50,  //  heartbeat 間隔（60秒以下にすること）
        ],
    ]
];
```

## コネクションプールについて

* 各プロセスは独自のコネクションプールを持ち、プロセス間では共有されません。
* コルーチン無効時は処理が順次実行されるため、接続数は最大 1 つです。
* コルーチン有効時は処理が並列実行され、プールは `min_connections` ～ `max_connections` の間で動的に調整されます。
* Redis を操作するコルーチン数が `max_connections` を超えると、超過分は最大 `wait_timeout` 秒待機し、超えると例外が発生します。
* アイドル時（コルーチン有無問わず）、接続は `idle_timeout` 経過後に解放され、`min_connections`（0 可）になるまで減ります。

## サンプル

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

## Redis インターフェース

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

以下と同等です。

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
> `Redis::select($db)` の使用には注意してください。webman は常駐メモリのフレームワークなので、あるリクエストで `Redis::select($db)` でデータベースを切り替えると、後続のリクエストに影響します。複数データベースを使う場合は、異なる `$db` を別々の Redis 接続として設定することを推奨します。

## 複数の Redis 接続を使用する

設定ファイル `config/redis.php` の例：

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

デフォルトでは `default` の接続が使われます。`Redis::connection()` で Redis 接続を切り替えられます。

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## クラスタ設定

Redis クラスタを使う場合は、設定ファイルで `clusters` キーを使って定義します：

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

デフォルトではノード上でクライアント側シャーディングが行われ、ノードプールと大量のメモリを確保できます。クライアント側シャーディングはフェイルオーバーを扱わないため、主に別の主 DB からのキャッシュ用です。Redis ネイティブクラスタを使う場合は、設定の `options` で次を指定します：

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

## パイプラインコマンド

一度に多数のコマンドを送る場合は、パイプラインが推奨されます。`pipeline` メソッドは Redis インスタンスのクロージャを受け取り、渡したコマンドは一括で実行されます：

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
