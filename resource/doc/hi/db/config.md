# डेटाबेस कॉन्फ़िगरेशन (Laravel शैली)
webman/database निम्नलिखित डेटाबेस और संस्करणों का समर्थन करता है:

- MySQL 5.6+
- PostgreSQL 9.4+
- SQLite 3.8.8+
- SQL Server 2017+

कॉन्फ़िगरेशन फ़ाइल `config/database.php` में स्थित है।

```php
return [
    // डिफ़ॉल्ट डेटाबेस
    'default' => 'mysql',
    // विभिन्न डेटाबेस कॉन्फ़िगरेशन
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
            'pool' => [ // कनेक्शन पूल कॉन्फ़िगरेशन, केवल swoole/swow ड्राइवरों के साथ समर्थित
                'max_connections' => 5, // अधिकतम कनेक्शन संख्या
                'min_connections' => 1, // न्यूनतम कनेक्शन संख्या
                'wait_timeout' => 3,    // पूल से कनेक्शन प्राप्त करने की अधिकतम प्रतीक्षा अवधि, टाइमआउट पर एक्सेप्शन फेंकेगा
                'idle_timeout' => 60,   // पूल में कनेक्शन की अधिकतम निष्क्रियता अवधि, टाइमआउट पर min_connections तक वापस ले लिया जाएगा
                'heartbeat_interval' => 50, // हार्टबीट अंतराल (सेकंड में), 60 से कम की सिफारिश
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

## कई डेटाबेस का उपयोग करना
उपयोग किए जाने वाले डेटाबेस का चयन करने के लिए `Db::connection('कॉन्फ़िग_नाम')` का उपयोग करें, जहाँ `कॉन्फ़िग_नाम` `config/database.php` फ़ाइल में संबंधित कॉन्फ़िगरेशन का `key` है।

उदाहरण निम्नलिखित कॉन्फ़िगरेशन के साथ:

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

डेटाबेस बदलने के लिए:
```php
// डिफ़ॉल्ट डेटाबेस का उपयोग करें, Db::connection('mysql')->table('users')->where('name', 'John')->first(); के बराबर
$users = Db::table('users')->where('name', 'John')->first(); 
// mysql2 का उपयोग करें
$users = Db::connection('mysql2')->table('users')->where('name', 'John')->first();
// pgsql का उपयोग करें
$users = Db::connection('pgsql')->table('users')->where('name', 'John')->first();
```
