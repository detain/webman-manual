# Configuration de la base de données (style Laravel)
webman/database prend en charge les bases de données et versions suivantes :

- MySQL 5.6+
- PostgreSQL 9.4+
- SQLite 3.8.8+
- SQL Server 2017+

Le fichier de configuration se trouve dans `config/database.php`.

```php
return [
    // Base de données par défaut
    'default' => 'mysql',
    // Diverses configurations de bases de données
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
            'pool' => [ // Configuration du pool de connexions, uniquement avec les pilotes swoole/swow
                'max_connections' => 5, // Nombre maximal de connexions
                'min_connections' => 1, // Nombre minimal de connexions
                'wait_timeout' => 3,    // Délai maximal d'attente pour obtenir une connexion, lance une exception en cas de dépassement
                'idle_timeout' => 60,   // Durée maximale d'inactivité des connexions dans le pool, récupération au dépassement jusqu'à min_connections
                'heartbeat_interval' => 50, // Intervalle de pulsation en secondes, recommandé inférieur à 60
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

## Utiliser plusieurs bases de données
Utilisez `Db::connection('nom_config')` pour sélectionner la base de données, où `nom_config` correspond à la `key` dans le fichier `config/database.php`.

Exemple avec la configuration suivante :

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

Pour changer de base de données :
```php
// Utiliser la base par défaut, équivalent à Db::connection('mysql')->table('users')->where('name', 'John')->first();
$users = Db::table('users')->where('name', 'John')->first(); 
// Utiliser mysql2
$users = Db::connection('mysql2')->table('users')->where('name', 'John')->first();
// Utiliser pgsql
$users = Db::connection('pgsql')->table('users')->where('name', 'John')->first();
```
