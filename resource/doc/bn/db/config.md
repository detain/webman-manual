# ডাটাবেস কনফিগারেশন (Laravel স্টাইল)
webman/database নিম্নলিখিত ডাটাবেস এবং সংস্করণ সমর্থন করে:

- MySQL 5.6+
- PostgreSQL 9.4+
- SQLite 3.8.8+
- SQL Server 2017+

কনফিগারেশন ফাইল `config/database.php` এ অবস্থিত।

```php
return [
    // ডিফল্ট ডাটাবেস
    'default' => 'mysql',
    // বিভিন্ন ডাটাবেস কনফিগারেশন
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
            'pool' => [ // সংযোগ পুল কনফিগারেশন, শুধুমাত্র swoole/swow ড্রাইভার সমর্থিত
                'max_connections' => 5, // সর্বোচ্চ সংযোগ সংখ্যা
                'min_connections' => 1, // সর্বনিম্ন সংযোগ সংখ্যা
                'wait_timeout' => 3,    // পুল থেকে সংযোগ নেওয়ার সর্বোচ্চ অপেক্ষার সময়, সময়সীমা অতিক্রম করলে এক্সেপশন নিক্ষেপ করবে
                'idle_timeout' => 60,   // পুলে সংযোগের সর্বোচ্চ নিষ্ক্রিয় সময়, সময়সীমা অতিক্রম করলে min_connections পর্যন্ত পুনরুদ্ধার হবে
                'heartbeat_interval' => 50, // হার্টবিট ব্যবধান (সেকেন্ডে), ৬০ এর কম সুপারিশকৃত
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

## একাধিক ডাটাবেস ব্যবহার
ব্যবহারের জন্য ডাটাবেস নির্বাচন করতে `Db::connection('কনফিগ_নাম')` ব্যবহার করুন, যেখানে `কনফিগ_নাম` হল `config/database.php` ফাইলের সংশ্লিষ্ট কনফিগারেশনের `key`।

উদাহরণ নিম্নলিখিত কনফিগারেশন সহ:

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

নিম্নরূপ ডাটাবেস পরিবর্তন করুন:
```php
// ডিফল্ট ডাটাবেস ব্যবহার করুন, Db::connection('mysql')->table('users')->where('name', 'John')->first(); এর সমতুল্য
$users = Db::table('users')->where('name', 'John')->first(); 
// mysql2 ব্যবহার করুন
$users = Db::connection('mysql2')->table('users')->where('name', 'John')->first();
// pgsql ব্যবহার করুন
$users = Db::connection('pgsql')->table('users')->where('name', 'John')->first();
```
