# 設定檔案

## 位置
webman 的設定檔案位於 `config/` 目錄下，專案中可以透過 `config()` 函數來取得對應的設定。

## 取得設定

取得所有設定
```php
config();
```

取得 `config/app.php` 裡的所有設定
```php
config('app');
```

取得 `config/app.php` 裡的 `debug` 設定
```php
config('app.debug');
```

若設定是陣列，可以透過 `.` 來取得陣列內部元素的值，例如
```php
config('file.key1.key2');
```

## 預設值
```php
config($key, $default);
```
config 透過第二個參數傳遞預設值，若設定不存在則回傳預設值。
設定不存在且沒有設定預設值則回傳 null。

## 自訂設定
開發者可以在 `config/` 目錄下新增自己的設定檔案，例如

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**取得設定時使用**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## 變更設定
webman 不支援動態修改設定，所有設定必須手動修改對應的設定檔案，並 reload 或 restart 重啟

> **注意**
> 伺服器設定 `config/server.php` 以及 process 設定 `config/process.php` 不支援 reload，需要 restart 重啟才能生效

## 特別提醒
若要在 config 下的子目錄建立設定檔案並讀取，例如：`config/order/status.php`，則 `config/order` 目錄下需要有一個 `app.php` 檔案，內容如下
```php
<?php
return [
    'enable' => true,
];
```
`enable` 為 `true` 代表讓框架讀取這個目錄的設定。
最終設定檔案目錄樹類似下面這樣
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
這樣你就可以透過 `config.order.status` 讀取 `status.php` 中回傳的陣列或特定 key 的資料了。


## 設定檔案說明

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // 監聽埠（從 1.6.0 版本開始移除，改在 config/process.php 中設定）
    'transport' => 'tcp', // 傳輸層協定（從 1.6.0 版本開始移除，改在 config/process.php 中設定）
    'context' => [], // ssl 等設定（從 1.6.0 版本開始移除，改在 config/process.php 中設定）
    'name' => 'webman', // process 名稱（從 1.6.0 版本開始移除，改在 config/process.php 中設定）
    'count' => cpu_count() * 4, // process 數量（從 1.6.0 版本開始移除，改在 config/process.php 中設定）
    'user' => '', // 使用者（從 1.6.0 版本開始移除，改在 config/process.php 中設定）
    'group' => '', // 使用者群組（從 1.6.0 版本開始移除，改在 config/process.php 中設定）
    'reusePort' => false, // 是否開啟埠複用（從 1.6.0 版本開始移除，改在 config/process.php 中設定）
    'event_loop' => '',  // 事件迴圈類別，預設自動選擇
    'stop_timeout' => 2, // 收到 stop/restart/reload 訊號時，等待處理完成的最大時間，超過此時間 process 未結束則強制結束
    'pid_file' => runtime_path() . '/webman.pid', // pid 檔案儲存位置
    'status_file' => runtime_path() . '/webman.status', // status 檔案儲存位置
    'stdout_file' => runtime_path() . '/logs/stdout.log', // 標準輸出檔案位置，webman 啟動後所有輸出都會寫入此檔案
    'log_file' => runtime_path() . '/logs/workerman.log', // workerman 日誌檔案位置
    'max_package_size' => 10 * 1024 * 1024 // 最大資料包大小，10M。上傳檔案大小受到此限制
];
```

#### app.php
```php
return [
    'debug' => true,  // 是否開啟 debug 模式，開啟後頁面報錯會輸出呼叫堆疊等除錯資訊，為安全起見生產環境應關閉 debug
    'error_reporting' => E_ALL, // 錯誤報告級別
    'default_timezone' => 'Asia/Shanghai', // 預設時區
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // public 目錄位置
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // runtime 目錄位置
    'controller_suffix' => 'Controller', // 控制器後綴
    'controller_reuse' => false, // 控制器是否複用
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // webman process 設定
    'webman' => [ 
        'handler' => Http::class, // process 處理類別
        'listen' => 'http://0.0.0.0:8787', // 監聽位址
        'count' => cpu_count() * 4, // process 數量，預設 cpu 的 4 倍
        'user' => '', // process 執行使用者，應使用低權限使用者
        'group' => '', // process 執行群組，應使用低權限群組
        'reusePort' => false, // 是否開啟 reusePort，開啟後連線會均勻分佈到不同的 worker process
        'eventLoop' => '', // 事件迴圈類別，為空時自動使用 server.event_loop 設定
        'context' => [], // 監聽上下文設定，例如 ssl
        'constructor' => [ // process 處理類別建構函數參數，本例為 Http 類別的建構函數參數
            'requestClass' => Request::class, // 可自訂請求類別
            'logger' => Log::channel('default'), // 日誌實例
            'appPath' => app_path(), // app 目錄位置
            'publicPath' => public_path() // public 目錄位置
        ]
    ],
    // 監控 process，用於偵測檔案更新自動載入和記憶體洩漏
    'monitor' => [
        'handler' => app\process\Monitor::class, // 處理類別
        'reloadable' => false, // 當前 process 不執行 reload
        'constructor' => [ // process 處理類別建構函數參數
            // 監聽的目錄，不要過多，會導致偵測變慢
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // 監聽這些副檔名檔案的更新
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // 其它選項
            'options' => [
                // 是否開啟檔案監控，僅在 linux 下有效，預設守護 process 模式不開啟檔案監控
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // 是否開啟記憶體監控，僅支援在 linux 下開啟
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// 回傳一個 psr-11 依賴注入容器實例
return new Webman\Container;
```

#### dependence.php
```php
// 用於設定依賴注入容器中的服務和依賴關係
return [];
```

#### route.php
```php

use support\Route;
// 定義 /test 路徑的路由
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
    'handler' => Raw::class // 預設視圖處理類別
];
```

### autoload.php
```php
// 設定框架自動載入的檔案
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
// 快取設定
return [
    'default' => 'file', // 預設檔案
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // 快取檔案儲存位置
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // redis 連線名稱，對應 redis.php 裡的設定
        ],
        'array' => [
            'driver' => 'array' // 記憶體快取，重啟後失效
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
 // 預設資料庫
 'default' => 'mysql',
 // 各種資料庫設定
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
    // 設定異常處理類別
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // 處理器
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // 日誌檔名
                    7, //$maxFiles // 保留 7 天內的日誌
                    Monolog\Logger::DEBUG, // 日誌級別
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // 格式化器
                    'constructor' => [null, 'Y-m-d H:i:s', true], // 格式化參數
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // 類型
    'type' => 'file', // or redis or redis_cluster
     // 處理器
    'handler' => FileSessionHandler::class,
     // 設定
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // 儲存目錄
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
    'session_name' => 'PHPSID', // session 名稱
    'auto_update_timestamp' => false, // 是否自動更新時間戳，避免 session 過期
    'lifetime' => 7*24*60*60, // 生命週期
    'cookie_lifetime' => 365*24*60*60, // cookie 生命週期
    'cookie_path' => '/', // cookie 路徑
    'domain' => '', // cookie 網域
    'http_only' => true, // 僅 http 存取
    'secure' => false, // 僅 https 存取
    'same_site' => '', // SameSite 屬性
    'gc_probability' => [1, 1000], // session 回收機率
];
```

#### middleware.php
```php
// 設定中介軟體
return [];
```

#### static.php
```php
return [
    'enable' => true, // 是否開啟 webman 的靜態檔案存取
    'middleware' => [ // 靜態檔案中介軟體，可用於設定快取策略、跨域等
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // 預設語言
    'locale' => 'zh_CN',
    // 回退語言
    'fallback_locale' => ['zh_CN', 'en'],
    // 語言檔案儲存位置
    'path' => base_path() . '/resource/translations',
];
```
