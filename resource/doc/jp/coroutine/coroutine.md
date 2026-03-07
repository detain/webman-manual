# コルーチン

webmanはworkermanをベースに開発されているため、webmanはworkermanのコルーチン機能を使用できます。
コルーチンは`Swoole`、`Swow`、`Fiber`の3つのドライバをサポートしています。

## 前提条件

- PHP >= 8.1
- Workerman >= 5.1.0 (`composer require workerman/workerman ~v5.1`)
- webman-framework >= 2.1 (`composer require workerman/webman-framework ~v2.1`)
- SwooleまたはSwow拡張のインストール、または`composer require revolt/event-loop`（Fiber用）
- コルーチンはデフォルトで無効。`eventLoop`を設定して有効にする必要がある

## 有効化方法

webmanはプロセスごとに異なるドライバを設定できるため、`config/process.php`（プラグインのprocess.php設定を含む）で`eventLoop`によりコルーチンドライバを設定できます：

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // デフォルトは空でSelectまたはEventを自動選択、コルーチン無効
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ],
    'my-coroutine' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        // コルーチン有効化には Workerman\Events\Swoole::class、Workerman\Events\Swow::class、Workerman\Events\Fiber::class のいずれかを設定
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ]
    
    // ... その他の設定は省略 ...
];
```

> **ヒント**
> webmanではプロセスごとに異なる`eventLoop`を設定できるため、特定のプロセスにのみコルーチンを有効にできます。
> 上記の例では8787ポートのサービスはコルーチン無効、8686ポートのサービスはコルーチン有効。nginxで転送すれば、コルーチンと非コルーチンの混在デプロイが可能です。

## コルーチン例

```php
<?php
namespace app\controller;

use support\Response;
use Workerman\Coroutine;
use Workerman\Timer;

class IndexController
{
    public function index(): Response
    {
        Coroutine::create(function(){
            Timer::sleep(1.5);
            echo "hello coroutine\n";
        });
        return response('hello webman');
    }

}
```

`eventLoop`が`Swoole`、`Swow`、`Fiber`の場合は、webmanは各リクエストごとにコルーチンを作成し、リクエスト処理中に新しいコルーチンを追加作成できます。

## コルーチンの制限

* Swoole、Swowをドライバとする場合、ブロッキングIOでコルーチンは自動的に切り替わり、同期コードを非同期実行できます。
* Fiberドライバでは、ブロッキングIO時にコルーチン切り替えは起きず、プロセスがブロックします。
* コルーチン使用時は、DB接続やファイル操作など同一リソースに対する複数コルーチンの同時操作は避けてください。リソース競合を防ぐには、接続プールやロックを使用してください。
* コルーチン使用時は、リクエストに関連する状態をグローバル変数や静的変数に保存しないでください。グローバル汚染の原因になります。代わりにコルーチンコンテキスト（`context`）で保存・取得してください。

## その他の注意事項

SwowはPHPのブロッキング関数をフックしますが、PHPの一部の挙動に影響するため、Swowを使用していないのにインストールしているとバグの原因になることがあります。

**推奨事項：**
* Swowを使用しないプロジェクトでは、Swow拡張をインストールしないでください
* Swowを使用するプロジェクトでは、`eventLoop`を`Workerman\Events\Swow::class`に設定してください

## コルーチンコンテキスト

コルーチン環境では、**リクエスト関連**の状態をグローバル変数や静的変数に保存しないでください。例：

```php
<?php

namespace app\controller;

use support\Request;
use Workerman\Timer;

class TestController
{
    protected static $name = '';

    public function index(Request $request)
    {
        static::$name = $request->get('name');
        Timer::sleep(5);
        return static::$name;
    }
}
```

> **注意**
> グローバル変数や静的変数の使用自体は禁止されていません。禁止されているのは**リクエスト関連の状態**をそこに保存することです。
> グローバル設定、DB接続、シングルトンなど、全体で共有すべきオブジェクトはグローバル変数や静的変数に保存して問題ありません。

プロセス数を1にして、以下の2リクエストを連続送信した場合：
http://127.0.0.1:8787/test?name=lilei
http://127.0.0.1:8787/test?name=hanmeimei

それぞれ`lilei`と`hanmeimei`が返る想定ですが、実際には両方とも`hanmeimei`になります。2番目のリクエストが静的変数`$name`を上書きし、1番目のリクエストがスリープから復帰した時点で既に`hanmeimei`になっているためです。

**正しくはcontextにリクエスト状態を保存する**

```php
<?php

namespace app\controller;

use support\Request;
use support\Context;
use Workerman\Timer;

class TestController
{
    public function index(Request $request)
    {
        Context::set('name', $request->get('name'));
        Timer::sleep(5);
        return Context::get('name');
    }
}
```

`support\Context`はコルーチンコンテキストを保存します。コルーチン終了時に関連するcontextデータは自動的に削除されます。
コルーチン環境では各リクエストが別コルーチンで実行されるため、リクエスト完了時にcontextが自動で破棄されます。
非コルーチン環境では、リクエスト終了時にcontextが破棄されます。

**ローカル変数はデータ汚染しない**

```php
<?php

namespace app\controller;

use support\Request;
use support\Context;
use Workerman\Timer;

class TestController
{
    public function index(Request $request)
    {
        $name = $request->get('name');
        Timer::sleep(5);
        return $name;
    }
}
```

`$name`はローカル変数のため、コルーチン間で共有されず、コルーチンセーフです。

## Locker（ロック）

一部のコンポーネントや処理がコルーチンを想定しておらず、リソース競合や原子性の問題が起きる場合があります。このようなときは`Workerman\Locker`でロックし、並行実行を直列化して問題を防ぎます。

```php
<?php

namespace app\controller;

use Redis;
use support\Response;
use Workerman\Coroutine\Locker;

class IndexController
{
    public function index(): Response
    {
        static $redis;
        if (!$redis) {
            $redis = new Redis();
            $redis->connect('127.0.0.1', 6379);
        }
        // ロックなしだと、Swooleでは "Socket#10 has already been bound to another coroutine#10" のようなエラーが発生する可能性
        // Swowではcoredumpの可能性
        // FiberではRedis拡張が同期ブロッキングIOのため問題なし
        Locker::lock('redis');
        $time = $redis->time();
        Locker::unlock('redis');
        return json($time);
    }

}
```

## Parallel（並列実行）

複数タスクを並列実行して結果を集めるには、`Workerman\Parallel`を使用します。

```php
<?php

namespace app\controller;

use support\Response;
use Workerman\Coroutine\Parallel;

class IndexController
{
    public function index(): Response
    {
        $parallel = new Parallel();
        for ($i=1; $i<5; $i++) {
            $parallel->add(function () use ($i) {
                // 何か処理
                return $i;
            });
        }
        $results = $parallel->wait();
        return json($results); // レスポンス: [1,2,3,4]
    }

}
```

## Pool（接続プール）

複数コルーチンで同一接続を共有するとデータが混在するため、DB・Redisなどの接続は接続プールで管理する必要があります。

webmanは [webman/database](../db/tutorial.md)、[webman/redis](../db/redis.md)、[webman/cache](../db/cache.md)、[webman/think-orm](../db/thinkorm.md)、[webman/think-cache](../db/thinkcache.md) などを提供しており、いずれも接続プールを内蔵し、コルーチン・非コルーチン両方で利用できます。

接続プールを持たないコンポーネントを組み込む場合は、`Workerman\Pool`を使用できます。例は以下の通りです。

**データベースコンポーネント**

```php
<?php
namespace app;

use Workerman\Coroutine\Context;
use Workerman\Coroutine;
use Workerman\Coroutine\Pool;

class Db
{
    private static ?Pool $pool = null;

    public static function __callStatic($name, $arguments)
    {
        if (self::$pool === null) {
            self::initializePool();
        }
        // コルーチンコンテキストから接続を取得し、同一コルーチンで同一接続を使用
        $pdo = Context::get('pdo');
        if (!$pdo) {
            $pdo = self::$pool->get();
            Context::set('pdo', $pdo);
            Coroutine::defer(function () use ($pdo) {
                self::$pool->put($pdo);
            });
        }
        return call_user_func_array([$pdo, $name], $arguments);
    }

    private static function initializePool(): void
    {
        self::$pool = new Pool(10);
        self::$pool->setConnectionCreator(function () {
            return new \PDO('mysql:host=127.0.0.1;dbname=your_database', 'your_username', 'your_password');
        });
        self::$pool->setConnectionCloser(function ($pdo) {
            $pdo = null;
        });
        self::$pool->setHeartbeatChecker(function ($pdo) {
            $pdo->query('SELECT 1');
        });
    }

}
```

**使用例**

```php
<?php
namespace app\controller;

use support\Response;
use app\Db;

class IndexController
{
    public function index(): Response
    {
        $value = Db::query('SELECT NOW() as now')->fetchAll();
        return json($value); // [{"now":"2025-02-06 23:41:03","0":"2025-02-06 23:41:03"}]
    }

}
```

## コルーチンと関連コンポーネントの詳細

[workerman コルーチンドキュメント](https://www.workerman.net/doc/workerman/coroutine/coroutine.html)を参照してください。

## コルーチンと非コルーチンの混在デプロイ

webmanはコルーチンと非コルーチンの混在デプロイをサポートしています。例えば、通常の処理は非コルーチン、遅いIOの処理はコルーチンで行い、nginxでリクエストを振り分けます。

例 `config/process.php`

```php
return [
    'webman' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8787',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => '', // デフォルトは空、コルーチン無効
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ],
    'my-coroutine' => [
        'handler' => Http::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 1,
        'user' => '',
        'group' => '',
        'reusePort' => false,
        'eventLoop' => Workerman\Events\Swoole::class,
        'context' => [],
        'constructor' => [
            'requestClass' => Request::class,
            'logger' => Log::channel('default'),
            'appPath' => app_path(),
            'publicPath' => public_path()
        ]
    ],
    
    // ... その他の設定は省略 ...
];
```

nginxでリクエストを振り分けます：

```nginx
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # /tastで始まるリクエストを8686ポートへ。必要に応じて/tastを変更してください
  location /tast {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # その他は8787ポートへ
  location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      if (!-f $request_filename){
          proxy_pass http://webman;
      }
  }
}
```
