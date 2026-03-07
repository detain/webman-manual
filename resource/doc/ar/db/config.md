# تكوين قاعدة البيانات (أسلوب Laravel)
يدعم webman/database قواعد البيانات والإصدارات التالية:

- MySQL 5.6+
- PostgreSQL 9.4+
- SQLite 3.8.8+
- SQL Server 2017+

يوجد ملف التكوين في `config/database.php`.

```php
return [
    // قاعدة البيانات الافتراضية
    'default' => 'mysql',
    // تكوينات قواعد البيانات المختلفة
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
            'pool' => [ // تكوين تجمع الاتصالات، يدعم فقط سائقي swoole/swow
                'max_connections' => 5, // الحد الأقصى للاتصالات
                'min_connections' => 1, // الحد الأدنى للاتصالات
                'wait_timeout' => 3,    // الحد الأقصى لوقت انتظار الحصول على اتصال، يرمي استثناء عند التجاوز
                'idle_timeout' => 60,   // الحد الأقصى لوقت الخمول للاتصالات في التجمع، استرجاع عند التجاوز حتى min_connections
                'heartbeat_interval' => 50, // فترة نبض القلب بالثواني، يُنصح بأقل من 60
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

## استخدام عدة قواعد بيانات
استخدم `Db::connection('اسم_التكوين')` لاختيار قاعدة البيانات، حيث `اسم_التكوين` هو الـ `key` المقابل في ملف `config/database.php`.

مثال مع التكوين التالي:

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

للتبديل بين قواعد البيانات:
```php
// استخدام القاعدة الافتراضية، يعادل Db::connection('mysql')->table('users')->where('name', 'John')->first();
$users = Db::table('users')->where('name', 'John')->first(); 
// استخدام mysql2
$users = Db::connection('mysql2')->table('users')->where('name', 'John')->first();
// استخدام pgsql
$users = Db::connection('pgsql')->table('users')->where('name', 'John')->first();
```
