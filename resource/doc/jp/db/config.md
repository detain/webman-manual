# データベース設定（Laravel スタイル）
webman/database がサポートするデータベースとバージョンは以下の通りです：

- MySQL 5.6+
- PostgreSQL 9.4+
- SQLite 3.8.8+
- SQL Server 2017+

データベースの設定ファイルは `config/database.php` にあります。

```php
return [
    // デフォルトのデータベース
    'default' => 'mysql',
    // 各種データベースの設定
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
            'pool' => [ // コネクションプール設定、swoole/swow ドライバのみ対応
                'max_connections' => 5, // 最大接続数
                'min_connections' => 1, // 最小接続数
                'wait_timeout' => 3,    // プールから接続を取得する最大待機時間、タイムアウトで例外をスロー
                'idle_timeout' => 60,   // プール内接続の最大アイドル時間、タイムアウトで回収、min_connections まで減少
                'heartbeat_interval' => 50, // ハートビート間隔（秒）、60秒未満を推奨
            ],
         ],
         
         'sqlite' => [
             'driver'   => 'sqlite',
             'database' => '',
             'prefix'   => '',
             'pool' => [ // コネクションプール設定、swoole/swow ドライバのみ対応
                'max_connections' => 5, // 最大接続数
                'min_connections' => 1, // 最小接続数
                'wait_timeout' => 3,    // プールから接続を取得する最大待機時間、タイムアウトで例外をスロー
                'idle_timeout' => 60,   // プール内接続の最大アイドル時間、タイムアウトで回収、min_connections まで減少
                'heartbeat_interval' => 50, // ハートビート間隔（秒）、60秒未満を推奨
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
             'pool' => [ // コネクションプール設定、swoole/swow ドライバのみ対応
                'max_connections' => 5, // 最大接続数
                'min_connections' => 1, // 最小接続数
                'wait_timeout' => 3,    // プールから接続を取得する最大待機時間、タイムアウトで例外をスロー
                'idle_timeout' => 60,   // プール内接続の最大アイドル時間、タイムアウトで回収、min_connections まで減少
                'heartbeat_interval' => 50, // ハートビート間隔（秒）、60秒未満を推奨
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
             'pool' => [ // コネクションプール設定、swoole/swow ドライバのみ対応
                'max_connections' => 5, // 最大接続数
                'min_connections' => 1, // 最小接続数
                'wait_timeout' => 3,    // プールから接続を取得する最大待機時間、タイムアウトで例外をスロー
                'idle_timeout' => 60,   // プール内接続の最大アイドル時間、タイムアウトで回収、min_connections まで減少
                'heartbeat_interval' => 50, // ハートビート間隔（秒）、60秒未満を推奨
            ],
         ],
     ],
 ];
```

## 複数のデータベースの使用
使用するデータベースを選択するには `Db::connection('設定名')` を使用します。`設定名` は設定ファイル `config/database.php` 内の対応する設定の `key` です。

例えば、以下のようなデータベース設定の場合：

```php
 return [
     // デフォルトのデータベース
     'default' => 'mysql',
     // 各種データベースの設定
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

このようにデータベースを切り替えます。
```php
// デフォルトのデータベースを使用、Db::connection('mysql')->table('users')->where('name', 'John')->first(); と同等
$users = Db::table('users')->where('name', 'John')->first(); 
// mysql2 を使用
$users = Db::connection('mysql2')->table('users')->where('name', 'John')->first();
// pgsql を使用
$users = Db::connection('pgsql')->table('users')->where('name', 'John')->first();
```
