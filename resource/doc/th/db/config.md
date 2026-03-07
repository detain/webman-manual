# การกำหนดค่าฐานข้อมูล (สไตล์ Laravel)
webman/database รองรับฐานข้อมูลและเวอร์ชันดังนี้:

- MySQL 5.6+
- PostgreSQL 9.4+
- SQLite 3.8.8+
- SQL Server 2017+

ไฟล์การกำหนดค่าอยู่ที่ `config/database.php`

```php
return [
    // ฐานข้อมูลเริ่มต้น
    'default' => 'mysql',
    // การกำหนดค่าฐานข้อมูลต่างๆ
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
            'pool' => [ // การตั้งค่าพูลการเชื่อมต่อ รองรับเฉพาะไดรเวอร์ swoole/swow
                'max_connections' => 5, // จำนวนการเชื่อมต่อสูงสุด
                'min_connections' => 1, // จำนวนการเชื่อมต่อขั้นต่ำ
                'wait_timeout' => 3,    // เวลารอสูงสุดในการได้การเชื่อมต่อ จะ throw exception เมื่อเกิน
                'idle_timeout' => 60,   // เวลาว่างสูงสุดของการเชื่อมต่อในพูล จะคืนเมื่อเกินจนถึง min_connections
                'heartbeat_interval' => 50, // ช่วงหัวใจ (วินาที) แนะนำน้อยกว่า 60
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

## ใช้หลายฐานข้อมูล
ใช้ `Db::connection('ชื่อการตั้งค่า')` เพื่อเลือกฐานข้อมูล โดย `ชื่อการตั้งค่า` คือ `key` ที่ตรงกันในไฟล์ `config/database.php`

ตัวอย่างการตั้งค่าดังนี้:

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

สลับฐานข้อมูลดังนี้:
```php
// ใช้ฐานข้อมูลเริ่มต้น เทียบเท่า Db::connection('mysql')->table('users')->where('name', 'John')->first();
$users = Db::table('users')->where('name', 'John')->first(); 
// ใช้ mysql2
$users = Db::connection('mysql2')->table('users')->where('name', 'John')->first();
// ใช้ pgsql
$users = Db::connection('pgsql')->table('users')->where('name', 'John')->first();
```
