# Cấu hình cơ sở dữ liệu (phong cách Laravel)
webman/database hỗ trợ các cơ sở dữ liệu và phiên bản sau:

- MySQL 5.6+
- PostgreSQL 9.4+
- SQLite 3.8.8+
- SQL Server 2017+

Tệp cấu hình nằm tại `config/database.php`.

```php
return [
    // Cơ sở dữ liệu mặc định
    'default' => 'mysql',
    // Các cấu hình cơ sở dữ liệu khác nhau
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
            'pool' => [ // Cấu hình pool kết nối, chỉ hỗ trợ trình điều khiển swoole/swow
                'max_connections' => 5, // Số kết nối tối đa
                'min_connections' => 1, // Số kết nối tối thiểu
                'wait_timeout' => 3,    // Thời gian chờ tối đa khi lấy kết nối từ pool, vượt quá sẽ ném ngoại lệ
                'idle_timeout' => 60,   // Thời gian nhàn rỗi tối đa của kết nối trong pool, vượt quá sẽ thu hồi đến min_connections
                'heartbeat_interval' => 50, // Khoảng thời gian kiểm tra (giây), khuyến nghị nhỏ hơn 60
            ],
         ],
         
         'sqlite' => [
             'driver'   => 'sqlite',
             'database' => '',
             'prefix'   => '',
             'pool' => [
                'max_connections' => 5,
                'min_connections' => 1,
                'wait_timeout' => 3,
                'idle_timeout' => 60,
                'heartbeat_interval' => 50,
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
             'pool' => [
                'max_connections' => 5,
                'min_connections' => 1,
                'wait_timeout' => 3,
                'idle_timeout' => 60,
                'heartbeat_interval' => 50,
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
             'pool' => [
                'max_connections' => 5,
                'min_connections' => 1,
                'wait_timeout' => 3,
                'idle_timeout' => 60,
                'heartbeat_interval' => 50,
            ],
         ],
     ],
 ];
```

## Sử dụng nhiều cơ sở dữ liệu
Dùng `Db::connection('tên_cấu_hình')` để chọn cơ sở dữ liệu, trong đó `tên_cấu_hình` là `key` tương ứng trong tệp `config/database.php`.

Ví dụ với cấu hình sau:

```php
 return [
     'default' => 'mysql',
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

Chuyển đổi cơ sở dữ liệu như sau:
```php
// Dùng cơ sở mặc định, tương đương Db::connection('mysql')->table('users')->where('name', 'John')->first();
$users = Db::table('users')->where('name', 'John')->first(); 
// Dùng mysql2
$users = Db::connection('mysql2')->table('users')->where('name', 'John')->first();
// Dùng pgsql
$users = Db::connection('pgsql')->table('users')->where('name', 'John')->first();
```
