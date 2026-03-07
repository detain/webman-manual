# 配置資料庫（Laravel 風格）
webman/database 支援的資料庫及版本如下：

 - MySQL 5.6+ 
 - PostgreSQL 9.4+ 
 - SQLite 3.8.8+
 - SQL Server 2017+ 
 
 資料庫設定檔位於 `config/database.php`。
 
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
             'pool' => [ // 連接池設定，僅支援 swoole/swow 驅動
                'max_connections' => 5, // 最大連接數
                'min_connections' => 1, // 最小連接數
                'wait_timeout' => 3,    // 從連接池取得連接的最大等候時間，逾時會拋出異常
                'idle_timeout' => 60,   // 連接池中連接的最大閒置時間，逾時會關閉回收，直到連接數為 min_connections
                'heartbeat_interval' => 50, // 連接池心跳檢測間隔，單位秒，建議小於 60 秒
            ],
         ],
         
         'sqlite' => [
             'driver'   => 'sqlite',
             'database' => '',
             'prefix'   => '',
             'pool' => [ // 連接池設定，僅支援 swoole/swow 驅動
                'max_connections' => 5, // 最大連接數
                'min_connections' => 1, // 最小連接數
                'wait_timeout' => 3,    // 從連接池取得連接的最大等候時間，逾時會拋出異常
                'idle_timeout' => 60,   // 連接池中連接的最大閒置時間，逾時會關閉回收，直到連接數為 min_connections
                'heartbeat_interval' => 50, // 連接池心跳檢測間隔，單位秒，建議小於 60 秒
            ],
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
             'pool' => [ // 連接池設定，僅支援 swoole/swow 驅動
                'max_connections' => 5, // 最大連接數
                'min_connections' => 1, // 最小連接數
                'wait_timeout' => 3,    // 從連接池取得連接的最大等候時間，逾時會拋出異常
                'idle_timeout' => 60,   // 連接池中連接的最大閒置時間，逾時會關閉回收，直到連接數為 min_connections
                'heartbeat_interval' => 50, // 連接池心跳檢測間隔，單位秒，建議小於 60 秒
            ],
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
             'pool' => [ // 連接池設定，僅支援 swoole/swow 驅動
                'max_connections' => 5, // 最大連接數
                'min_connections' => 1, // 最小連接數
                'wait_timeout' => 3,    // 從連接池取得連接的最大等候時間，逾時會拋出異常
                'idle_timeout' => 60,   // 連接池中連接的最大閒置時間，逾時會關閉回收，直到連接數為 min_connections
                'heartbeat_interval' => 50, // 連接池心跳檢測間隔，單位秒，建議小於 60 秒
            ],
         ],
     ],
 ];
 ```

 ## 使用多個資料庫
透過 `Db::connection('設定名')` 選擇要使用的資料庫，其中 `設定名` 為設定檔 `config/database.php` 中對應設定的 `key`。
 
 例如以下資料庫設定：

```php
 return [
     // 預設資料庫
     'default' => 'mysql',
     // 各種資料庫設定
     'connections' => [
 
         'mysql' => [
             'driver'      => 'mysql',
             'host'        =>   '127.0.0.1',
             'port'        => 3306,
             'database'    => 'webman',
             'username'    => 'webman',
             'password'    => '',
             'unix_socket' =>  '',
             'charset'     => 'utf8',
             'collation'   => 'utf8_unicode_ci',
             'prefix'      => '',
             'strict'      => true,
             'engine'      => null,
         ],
         
         'mysql2' => [
              'driver'      => 'mysql',
              'host'        => '127.0.0.1',
              'port'        => 3306,
              'database'    => 'webman2',
              'username'    => 'webman2',
              'password'    => '',
              'unix_socket' => '',
              'charset'     => 'utf8',
              'collation'   => 'utf8_unicode_ci',
              'prefix'      => '',
              'strict'      => true,
              'engine'      => null,
         ],
         'pgsql' => [
              'driver'   => 'pgsql',
              'host'     => '127.0.0.1',
              'port'     =>  5432,
              'database' => 'webman',
              'username' =>  'webman',
              'password' => '',
              'charset'  => 'utf8',
              'prefix'   => '',
              'schema'   => 'public',
              'sslmode'  => 'prefer',
          ],
 ];
```

像這樣切換資料庫。
```php
// 使用預設資料庫，等同於 Db::connection('mysql')->table('users')->where('name', 'John')->first();
$users = Db::table('users')->where('name', 'John')->first(); 
// 使用 mysql2
$users = Db::connection('mysql2')->table('users')->where('name', 'John')->first();
// 使用 pgsql
$users = Db::connection('pgsql')->table('users')->where('name', 'John')->first();
```
