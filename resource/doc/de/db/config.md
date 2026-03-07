# Datenbankkonfiguration (Laravel-Stil)
webman/database unterstützt folgende Datenbanken und Versionen:

- MySQL 5.6+
- PostgreSQL 9.4+
- SQLite 3.8.8+
- SQL Server 2017+

Die Konfigurationsdatei der Datenbank befindet sich unter `config/database.php`.

```php
return [
    // Standarddatenbank
    'default' => 'mysql',
    // Verschiedene Datenbankkonfigurationen
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
            'pool' => [ // Verbindungspool-Konfiguration, nur für swoole/swow-Treiber
                'max_connections' => 5, // Maximale Verbindungsanzahl
                'min_connections' => 1, // Minimale Verbindungsanzahl
                'wait_timeout' => 3,    // Maximale Wartezeit beim Abrufen einer Verbindung, bei Überschreitung wird eine Exception geworfen
                'idle_timeout' => 60,   // Maximale Leerlaufzeit der Verbindungen im Pool, bei Überschreitung werden sie zurückgegeben bis min_connections erreicht ist
                'heartbeat_interval' => 50, // Herzschlag-Intervall in Sekunden, empfohlen unter 60
            ],
         ],
         
         'sqlite' => [
             'driver'   => 'sqlite',
             'database' => '',
             'prefix'   => '',
             'pool' => [ // Verbindungspool-Konfiguration, nur für swoole/swow-Treiber
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
             'pool' => [ // Verbindungspool-Konfiguration, nur für swoole/swow-Treiber
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
             'pool' => [ // Verbindungspool-Konfiguration, nur für swoole/swow-Treiber
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

## Mehrere Datenbanken verwenden
Verwenden Sie `Db::connection('Konfigurationsname')`, um die zu verwendende Datenbank auszuwählen. Dabei ist `Konfigurationsname` der entsprechende `key` in der Konfigurationsdatei `config/database.php`.

Beispiel mit folgender Datenbankkonfiguration:

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

So wechseln Sie zwischen Datenbanken:
```php
// Standarddatenbank verwenden, entspricht Db::connection('mysql')->table('users')->where('name', 'John')->first();
$users = Db::table('users')->where('name', 'John')->first(); 
// mysql2 verwenden
$users = Db::connection('mysql2')->table('users')->where('name', 'John')->first();
// pgsql verwenden
$users = Db::connection('pgsql')->table('users')->where('name', 'John')->first();
```
