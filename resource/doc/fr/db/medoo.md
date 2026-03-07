# Base de données Medoo

[webman/medoo](https://github.com/webman-php/medoo) étend [Medoo](https://medoo.in/) avec un pool de connexions et fonctionne en environnement coroutine et non-coroutine. L'utilisation est identique à Medoo.

## Installation
`composer require webman/medoo`

## Configuration de la base de données Medoo
Fichier de configuration : `config/plugin/webman/medoo/database.php`

## Utilisation de la base de données Medoo
```php
<?php
namespace app\controller;

use support\Request;
use support\Medoo;

class Index
{
    public function index(Request $request)
    {
        $user = Medoo::get('user', '*', ['uid' => 1]);
        return json($user);
    }
}
```

> **Remarque**
> `Medoo::get('user', '*', ['uid' => 1]);`
> équivaut à
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## Configuration de plusieurs bases de données Medoo

**Configuration**
Ajoutez une nouvelle configuration dans `config/plugin/webman/medoo/database.php` avec une clé quelconque ; ici nous utilisons `other`.

```php
<?php
return [
    'default' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [ // Configuration du pool de connexions
            'max_connections' => 5, // Nombre maximal de connexions
            'min_connections' => 1, // Nombre minimal de connexions
            'wait_timeout' => 60,   // Délai d'attente maximal pour obtenir une connexion ; exception si dépassé
            'idle_timeout' => 3,    // Durée maximale d'inactivité des connexions dans le pool ; au-delà, elles sont fermées jusqu'à min_connections
            'heartbeat_interval' => 50, // Intervalle de pulsation du pool en secondes ; recommandé inférieur à 60 secondes
        ]
    ],
    // Ajout de la configuration 'other' ici
    'other' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [
            'max_connections' => 5,
            'min_connections' => 1,
            'wait_timeout' => 60,
            'idle_timeout' => 3,
            'heartbeat_interval' => 50,
        ],
    ],
];
```

## Utilisation de la base de données Medoo
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

Voir la [documentation officielle de Medoo](https://medoo.in/api/select)
