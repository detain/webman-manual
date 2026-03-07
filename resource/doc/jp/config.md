# 設定ファイル

## 位置
webmanの設定ファイルは`config/`ディレクトリにあり、プロジェクトでは`config()`関数を使用して対応する設定を取得できます。

## 設定の取得

すべての設定を取得する
```php
config();
```

`config/app.php`のすべての設定を取得する
```php
config('app');
```

`config/app.php`の`debug`設定を取得する
```php
config('app.debug');
```

設定が配列の場合、`.`を使って配列内の要素の値を取得できます。例えば
```php
config('file.key1.key2');
```

## デフォルト値
```php
config($key, $default);
```
2番目のパラメータでデフォルト値を渡し、設定が存在しない場合はデフォルト値を返します。
設定が存在せず、デフォルト値も設定されていない場合はnullを返します。

## カスタム設定
開発者は`config/`ディレクトリに独自の設定ファイルを追加できます。例えば

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**設定取得時に使用**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## 設定の変更
webmanは設定の動的変更をサポートしていません。すべての設定は対応する設定ファイルを手動で変更し、reloadまたはrestartを行う必要があります。

> **注意**
> サーバー設定`config/server.php`およびプロセス設定`config/process.php`はreloadをサポートしていません。restartが必要です。

## 特別な注意
`config/`のサブディレクトリに設定ファイルを作成して読み込む場合（例：`config/order/status.php`）、`config/order`ディレクトリに以下の内容の`app.php`ファイルが必要です
```php
<?php
return [
    'enable' => true,
];
```
`enable`を`true`にすると、フレームワークがこのディレクトリの設定を読み込みます。
設定ファイルのディレクトリ構造は以下のようになります
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
これで`config.order.status`で`status.php`の配列または特定のキーのデータを取得できます。


## 設定ファイルの説明

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // リスンポート（1.6.0以降削除、config/process.phpで設定）
    'transport' => 'tcp', // トランスポートプロトコル（1.6.0以降削除、config/process.phpで設定）
    'context' => [], // SSL等の設定（1.6.0以降削除、config/process.phpで設定）
    'name' => 'webman', // プロセス名（1.6.0以降削除、config/process.phpで設定）
    'count' => cpu_count() * 4, // プロセス数（1.6.0以降削除、config/process.phpで設定）
    'user' => '', // ユーザー（1.6.0以降削除、config/process.phpで設定）
    'group' => '', // ユーザーグループ（1.6.0以降削除、config/process.phpで設定）
    'reusePort' => false, // ポート再利用（1.6.0以降削除、config/process.phpで設定）
    'event_loop' => '',  // イベントループクラス、デフォルトで自動選択
    'stop_timeout' => 2, // stop/restart/reloadシグナル受信時の最大待機時間、超過すると強制終了
    'pid_file' => runtime_path() . '/webman.pid', // pidファイルの場所
    'status_file' => runtime_path() . '/webman.status', // statusファイルの場所
    'stdout_file' => runtime_path() . '/logs/stdout.log', // 標準出力ファイル、webman起動後の出力はここに書き込まれる
    'log_file' => runtime_path() . '/logs/workerman.log', // workermanログファイルの場所
    'max_package_size' => 10 * 1024 * 1024 // 最大パケットサイズ、10M。アップロードファイルサイズの制限
];
```

#### app.php
```php
return [
    'debug' => true,  // デバッグモード、有効時はエラー時にスタックトレース等を出力。本番環境では無効にすべき
    'error_reporting' => E_ALL, // エラー報告レベル
    'default_timezone' => 'Asia/Shanghai', // デフォルトタイムゾーン
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // publicディレクトリの場所
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // runtimeディレクトリの場所
    'controller_suffix' => 'Controller', // コントローラー接尾辞
    'controller_reuse' => false, // コントローラーの再利用
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // webmanプロセス設定
    'webman' => [ 
        'handler' => Http::class, // プロセスハンドラークラス
        'listen' => 'http://0.0.0.0:8787', // リスンアドレス
        'count' => cpu_count() * 4, // プロセス数、デフォルトはCPUの4倍
        'user' => '', // プロセス実行ユーザー、低権限ユーザーを使用すべき
        'group' => '', // プロセス実行グループ、低権限グループを使用すべき
        'reusePort' => false, // reusePort有効、有効時は接続が各workerに均等分散
        'eventLoop' => '', // イベントループクラス、空の場合はserver.event_loopを使用
        'context' => [], // リスンコンテキスト、ssl等
        'constructor' => [ // プロセスハンドラークラスのコンストラクタ引数、ここではHttpクラス
            'requestClass' => Request::class, // カスタムリクエストクラス
            'logger' => Log::channel('default'), // ロガーインスタンス
            'appPath' => app_path(), // appディレクトリの場所
            'publicPath' => public_path() // publicディレクトリの場所
        ]
    ],
    // ファイル更新検知とメモリリーク検出用のモニタープロセス
    'monitor' => [
        'handler' => app\process\Monitor::class, // ハンドラークラス
        'reloadable' => false, // このプロセスはreloadしない
        'constructor' => [ // プロセスハンドラークラスのコンストラクタ引数
            // 監視ディレクトリ、多すぎると検出が遅くなる
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // 更新を監視するファイル拡張子
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // その他オプション
            'options' => [
                // ファイル監視を有効化、Linuxのみ、デーモンモードではデフォルトで無効
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // メモリ監視を有効化、Linuxのみ
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// PSR-11依存性注入コンテナのインスタンスを返す
return new Webman\Container;
```

#### dependence.php
```php
// 依存性注入コンテナのサービスと依存関係を設定
return [];
```

#### route.php
```php

use support\Route;
// /testパスのルートを定義
Route::any('/test', function (Request $request) {
    return response('test');
});
```

#### view.php
```php
use support\view\Raw;
use support\view\Twig;
use support\view\Blade;
use support\view\ThinkPHP;

return [
    'handler' => Raw::class // デフォルトビューハンドラークラス
];
```

### autoload.php
```php
// フレームワークの自動読み込みファイルを設定
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php',
        base_path() . '/support/Response.php',
    ]
];
```

#### cache.php
```php
// キャッシュ設定
return [
    'default' => 'file', // デフォルトドライバ
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // キャッシュファイルの保存場所
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // redis接続名、redis.phpの設定に対応
        ],
        'array' => [
            'driver' => 'array' // メモリキャッシュ、再起動で消える
        ]
    ]
];
```

#### redis.php
```php
return [
    'default' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 0,
    ],
];
```

#### database.php
```php
return [
 // デフォルトデータベース
 'default' => 'mysql',
 // 各データベースの設定
 'connections' => [

     'mysql' => [
         'driver'      => 'mysql',
         'host'        => '127.0.0.1',
         'port'        => 3306,
         'database'    => 'webman',
         'username'    => 'webman',
         'password'    => '',
         'unix_socket' => '',
         'charset'     => 'utf8',
         'collation'   => 'utf8_unicode_ci',
         'prefix'      => '',
         'strict'      => true,
         'engine'      => null,
     ],

     'sqlite' => [
         'driver'   => 'sqlite',
         'database' => '',
         'prefix'   => '',
     ],

     'pgsql' => [
         'driver'   => 'pgsql',
         'host'     => '127.0.0.1',
         'port'     => 5432,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
         'schema'   => 'public',
         'sslmode'  => 'prefer',
     ],

     'sqlsrv' => [
         'driver'   => 'sqlsrv',
         'host'     => 'localhost',
         'port'     => 1433,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
     ],
 ],
];
```

#### exception.php
```php
return [
    // 例外ハンドラークラスを設定
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // ハンドラー
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // ログファイル名
                    7, //$maxFiles // 7日分のログを保持
                    Monolog\Logger::DEBUG, // ログレベル
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // フォーマッター
                    'constructor' => [null, 'Y-m-d H:i:s', true], // フォーマッターパラメータ
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // タイプ
    'type' => 'file', // or redis or redis_cluster
     // ハンドラー
    'handler' => FileSessionHandler::class,
     // 設定
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // 保存ディレクトリ
        ],
        'redis' => [
            'host' => '127.0.0.1',
            'port' => 6379,
            'auth' => '',
            'timeout' => 2,
            'database' => '',
            'prefix' => 'redis_session_',
        ],
        'redis_cluster' => [
            'host' => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth' => '',
            'prefix' => 'redis_session_',
        ]
    ],
    'session_name' => 'PHPSID', // セッション名
    'auto_update_timestamp' => false, // タイムスタンプ自動更新、セッション期限切れを防ぐ
    'lifetime' => 7*24*60*60, // 有効期限
    'cookie_lifetime' => 365*24*60*60, // cookie有効期限
    'cookie_path' => '/', // cookieパス
    'domain' => '', // cookieドメイン
    'http_only' => true, // HTTPのみ
    'secure' => false, // HTTPSのみ
    'same_site' => '', // SameSite属性
    'gc_probability' => [1, 1000], // セッションガベージコレクション確率
];
```

#### middleware.php
```php
// ミドルウェアを設定
return [];
```

#### static.php
```php
return [
    'enable' => true, // webmanの静的ファイル配信を有効化
    'middleware' => [ // 静的ファイルミドルウェア、キャッシュポリシー、CORS等に使用
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // デフォルト言語
    'locale' => 'zh_CN',
    // フォールバック言語
    'fallback_locale' => ['zh_CN', 'en'],
    // 言語ファイルの場所
    'path' => base_path() . '/resource/translations',
];
```
