# Veritabanı Yapılandırması (Laravel Tarzı)
webman/database aşağıdaki veritabanları ve sürümleri destekler:

- MySQL 5.6+
- PostgreSQL 9.4+
- SQLite 3.8.8+
- SQL Server 2017+

Yapılandırma dosyası `config/database.php` konumundadır.

```php
return [
    // Varsayılan veritabanı
    'default' => 'mysql',
    // Çeşitli veritabanı yapılandırmaları
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
            'pool' => [ // Bağlantı havuzu yapılandırması, yalnızca swoole/swow sürücüleriyle desteklenir
                'max_connections' => 5, // Maksimum bağlantı sayısı
                'min_connections' => 1, // Minimum bağlantı sayısı
                'wait_timeout' => 3,    // Havuzdan bağlantı alma için maksimum bekleme süresi, aşılınca istisna fırlatır
                'idle_timeout' => 60,   // Havuzdaki bağlantıların maksimum boşta kalma süresi, aşılınca min_connections'a kadar geri alınır
                'heartbeat_interval' => 50, // Nabız aralığı (saniye), 60'tan az önerilir
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

## Birden Fazla Veritabanı Kullanma
Kullanılacak veritabanını seçmek için `Db::connection('yapılandırma_adı')` kullanın. `yapılandırma_adı`, `config/database.php` dosyasındaki ilgili yapılandırmanın `key` değeridir.

Örnek yapılandırma:

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

Veritabanları arasında geçiş:
```php
// Varsayılan veritabanını kullan, Db::connection('mysql')->table('users')->where('name', 'John')->first(); ile eşdeğer
$users = Db::table('users')->where('name', 'John')->first(); 
// mysql2 kullan
$users = Db::connection('mysql2')->table('users')->where('name', 'John')->first();
// pgsql kullan
$users = Db::connection('pgsql')->table('users')->where('name', 'John')->first();
```
