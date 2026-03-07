# Démarrage rapide avec la base de données (composant Laravel)

[webman/database](https://github.com/webman-php/database) repose sur [illuminate/database](https://github.com/illuminate/database) et ajoute un pool de connexions pour les environnements avec et sans coroutines. L’usage est identique à Laravel.

Vous pouvez aussi consulter [Utiliser d’autres composants de base de données](others.md) pour utiliser ThinkPHP ou d’autres bases.

## Installation de la base de données

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

Un redémarrage est nécessaire après l’installation (reload ne suffit pas).

> **Astuce**
> webman/database dépend de `illuminate/database` de Laravel, les dépendances sont donc installées automatiquement.

> **Attention**
> Si vous n’avez pas besoin de pagination, d’événements ou de logs SQL, exécutez seulement :
> `composer require -W webman/database`

## Configuration de la base de données
`config/database.php`
```php

return [
    // Base de données par défaut
    'default' => 'mysql',

    // Connexions configurées
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'test',
            'username'    => 'root',
            'password'    => '',
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
            'options' => [
                PDO::ATTR_EMULATE_PREPARES => false, // Requis avec swoole ou swow
            ],
            'pool' => [ // Config du pool de connexions
                'max_connections' => 5, // Nombre max de connexions
                'min_connections' => 1, // Nombre min de connexions
                'wait_timeout' => 3,    // Délai max pour obtenir une connexion ; dépasse → exception. En environnement coroutines seulement
                'idle_timeout' => 60,   // Délai max d’inactivité des connexions ; ensuite récupération jusqu’à min_connections
                'heartbeat_interval' => 50, // Intervalle de heartbeat du pool en secondes ; < 60 s recommandé
            ],
        ],
    ],
];
```

À part `pool`, la config est la même que Laravel.

## À propos du pool de connexions
* Chaque processus a son propre pool ; les pools ne sont pas partagés entre processus.
* Sans coroutines, les requêtes s’exécutent séquentiellement, pas de concurrence, donc au plus une connexion dans le pool.
* Avec coroutines, les requêtes s’exécutent en parallèle ; le pool ajuste le nombre de connexions selon la charge, sans dépasser `max_connections` ni descendre sous `min_connections`.
* Comme le pool plafonne à `max_connections`, quand il y a plus de coroutines qui accèdent à la base, certaines attendent en queue jusqu’à `wait_timeout` secondes ; au-delà, une exception est levée.
* En inactivité (avec ou sans coroutines), les connexions sont récupérées après `idle_timeout` jusqu’à atteindre `min_connections` (`min_connections` peut être 0).


## Exemple d’utilisation
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        $default_uid = 29;
        $uid = $request->get('uid', $default_uid);
        $name = Db::table('users')->where('uid', $uid)->value('username');
        return response("hello $name");
    }
}
```

L’usage est le même que Laravel : la méthode `Db::table()` pour manipuler la base de données.
