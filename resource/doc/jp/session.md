# webman セッション管理

## 例
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

`$request->session();`で`Workerman\Protocols\Http\Session`インスタンスを取得し、インスタンスのメソッドでセッションデータの追加・変更・削除を行います。

> **注意**
> セッションオブジェクトが破棄されると、セッションデータは自動的に保存されます。
> セッションオブジェクトをグローバル変数に保存すると、オブジェクトが破棄されず自動保存されません。その場合は、手動で`$session->save()`を呼び出して保存してください。

## すべてのセッションデータを取得する
```php
$session = $request->session();
$all = $session->all();
```
配列が返されます。セッションデータがない場合は空の配列が返されます。


## セッション内の値を取得する
```php
$session = $request->session();
$name = $session->get('name');
```
データが存在しない場合はnullを返します。

getメソッドの第2引数にデフォルト値を渡すこともできます。セッション配列に対応する値が見つからない場合は、そのデフォルト値が返されます。例：
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## セッションを保存する
個別のデータを保存する場合はsetメソッドを使います。
```php
$session = $request->session();
$session->set('name', 'tom');
```
setには戻り値がありません。セッションオブジェクトが破棄されると、セッションは自動的に保存されます。

複数の値を保存する場合はputメソッドを使います。
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
同様に、putにも戻り値はありません。

## セッションデータを削除する
1件または複数件のセッションデータを削除する場合は`forget`メソッドを使います。
```php
$session = $request->session();
// 1件削除
$session->forget('name');
// 複数件削除
$session->forget(['name', 'age']);
```

また、deleteメソッドも提供されています。forgetとの違いは、deleteは1件しか削除できない点です。
```php
$session = $request->session();
// $session->forget('name'); と同等
$session->delete('name');
```

## セッションの値を取得して削除する
```php
$session = $request->session();
$name = $session->pull('name');
```
次のコードと同じ動作です：
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
該当するセッションが存在しない場合はnullを返します。


## すべてのセッションデータを削除する
```php
$request->session()->flush();
```
戻り値はありません。セッションオブジェクトが破棄されると、セッションは自動的にストレージから削除されます。


## セッションデータの存在を確認する
```php
$session = $request->session();
$has = $session->has('name');
```
該当するセッションが存在しない、または値がnullの場合はfalse、それ以外はtrueを返します。

```php
$session = $request->session();
$has = $session->exists('name');
```
上記もセッションデータの存在確認に使えます。違いは、セッションの値がnullの場合でもtrueを返す点です。

## ヘルパー関数 session()

webmanは同じ機能を実現するヘルパー関数`session()`を提供しています。
```php
// セッションインスタンスを取得
$session = session();
// 以下と同等
$session = $request->session();

// 値を取得
$value = session('key', 'default');
// 以下と同等
$value = session()->get('key', 'default');
// 以下と同等
$value = $request->session()->get('key', 'default');

// セッションに値を設定
session(['key1'=>'value1', 'key2' => 'value2']);
// 以下と同等
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// 以下と同等
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## 設定ファイル
セッションの設定ファイルは`config/session.php`にあり、内容は以下のようになっています。
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class または RedisSessionHandler::class または RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // handlerがFileSessionHandler::classの場合はfile、
    // handlerがRedisSessionHandler::classの場合はredis、
    // handlerがRedisClusterSessionHandler::classの場合はredis_cluster（Redisクラスタ）
    'type'    => 'file',

    // ハンドラごとに異なる設定を使用
    'config' => [
        // typeがfileの場合の設定
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // typeがredisの場合の設定
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // session_idを保存するクッキー名
    'auto_update_timestamp' => false,  // セッションの自動更新、デフォルトは無効
    'lifetime' => 7*24*60*60,          // セッション有効期限
    'cookie_lifetime' => 365*24*60*60, // session_idを保存するクッキーの有効期限
    'cookie_path' => '/',              // session_idを保存するクッキーのパス
    'domain' => '',                    // session_idを保存するクッキーのドメイン
    'http_only' => true,               // httpOnlyの有効化、デフォルトは有効
    'secure' => false,                 // HTTPSのみでセッションを有効にする、デフォルトは無効
    'same_site' => '',                 // CSRF攻撃とユーザー追跡の防止、指定値: strict/lax/none
    'gc_probability' => [1, 1000],     // セッションのガベージコレクション確率
];
```

## セキュリティ
セッションでは、クラスのインスタンスを直接保存するのは推奨されません。特に信頼できないソースのクラスインスタンスは、デシリアライズ時にセキュリティリスクを招く可能性があります。

