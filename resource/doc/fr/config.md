# Fichier de configuration

## Emplacement
Les fichiers de configuration de webman sont situés dans le répertoire `config/`. Dans le projet, vous pouvez utiliser la fonction `config()` pour accéder aux configurations correspondantes.

## Accéder aux configurations

Obtenir toutes les configurations :
```php
config();
```

Obtenir toutes les configurations dans `config/app.php` :
```php
config('app');
```

Obtenir la configuration `debug` dans `config/app.php` :
```php
config('app.debug');
```

Si la configuration est un tableau, vous pouvez utiliser `.` pour accéder aux valeurs imbriquées. Par exemple :
```php
config('file.key1.key2');
```

## Valeur par défaut
```php
config($key, $default);
```
Passez la valeur par défaut comme deuxième paramètre. Si la configuration n'existe pas, la valeur par défaut sera retournée. Si la configuration n'existe pas et qu'aucune valeur par défaut n'est définie, `null` est retourné.

## Configuration personnalisée
Les développeurs peuvent ajouter leurs propres fichiers de configuration dans le répertoire `config/`. Par exemple :

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**Utilisation lors de l'obtention de la configuration**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## Modifier les configurations
Webman ne prend pas en charge la modification dynamique des configurations. Toutes les configurations doivent être modifiées manuellement dans les fichiers correspondants, puis recharger (reload) ou redémarrer (restart).

> **Remarque**
> La configuration du serveur `config/server.php` et la configuration des processus `config/process.php` ne prennent pas en charge le rechargement (reload). Un redémarrage (restart) est nécessaire pour que les changements prennent effet.

## Rappel important
Si vous créez des fichiers de configuration dans un sous-répertoire de `config/`, par exemple `config/order/status.php`, vous avez besoin d'un fichier `app.php` dans le répertoire `config/order/` avec le contenu suivant :
```php
<?php
return [
    'enable' => true,
];
```
`enable` à `true` indique au framework de charger les configurations de ce répertoire.
La structure du répertoire de configuration doit ressembler à ceci :
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
Vous pouvez alors accéder au tableau ou aux clés spécifiques de `status.php` via `config.order.status`.


## Référence des fichiers de configuration

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // Port d'écoute (supprimé depuis 1.6.0, configuré dans config/process.php)
    'transport' => 'tcp', // Protocole de transport (supprimé depuis 1.6.0, configuré dans config/process.php)
    'context' => [], // SSL etc. (supprimé depuis 1.6.0, configuré dans config/process.php)
    'name' => 'webman', // Nom du processus (supprimé depuis 1.6.0, configuré dans config/process.php)
    'count' => cpu_count() * 4, // Nombre de processus (supprimé depuis 1.6.0, configuré dans config/process.php)
    'user' => '', // Utilisateur (supprimé depuis 1.6.0, configuré dans config/process.php)
    'group' => '', // Groupe (supprimé depuis 1.6.0, configuré dans config/process.php)
    'reusePort' => false, // Réutilisation du port (supprimé depuis 1.6.0, configuré dans config/process.php)
    'event_loop' => '',  // Classe de boucle d'événements, sélection automatique par défaut
    'stop_timeout' => 2, // Temps d'attente max à la réception du signal stop/restart/reload, forcer la sortie si dépassé
    'pid_file' => runtime_path() . '/webman.pid', // Emplacement du fichier PID
    'status_file' => runtime_path() . '/webman.status', // Emplacement du fichier de statut
    'stdout_file' => runtime_path() . '/logs/stdout.log', // Emplacement stdout, toute sortie après démarrage de webman
    'log_file' => runtime_path() . '/logs/workerman.log', // Emplacement du fichier log Workerman
    'max_package_size' => 10 * 1024 * 1024 // Taille max des paquets, 10M. Limite la taille des uploads
];
```

#### app.php
```php
return [
    'debug' => true,  // Mode debug, affiche la stack trace en cas d'erreur. Désactiver en production
    'error_reporting' => E_ALL, // Niveau de rapport d'erreurs
    'default_timezone' => 'Asia/Shanghai', // Fuseau horaire par défaut
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // Chemin du répertoire public
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // Chemin du répertoire runtime
    'controller_suffix' => 'Controller', // Suffixe du contrôleur
    'controller_reuse' => false, // Réutilisation des contrôleurs
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // Configuration des processus webman
    'webman' => [ 
        'handler' => Http::class, // Classe de gestionnaire du processus
        'listen' => 'http://0.0.0.0:8787', // Adresse d'écoute
        'count' => cpu_count() * 4, // Nombre de processus, 4x CPU par défaut
        'user' => '', // Utilisateur du processus, utiliser un utilisateur à faibles privilèges
        'group' => '', // Groupe du processus, utiliser un groupe à faibles privilèges
        'reusePort' => false, // Activer reusePort, distribue les connexions entre les workers
        'eventLoop' => '', // Classe de boucle d'événements, utilise server.event_loop si vide
        'context' => [], // Contexte d'écoute, p.ex. SSL
        'constructor' => [ // Paramètres du constructeur du gestionnaire, ici classe Http
            'requestClass' => Request::class, // Classe de requête personnalisée
            'logger' => Log::channel('default'), // Instance du logger
            'appPath' => app_path(), // Chemin du répertoire app
            'publicPath' => public_path() // Chemin du répertoire public
        ]
    ],
    // Processus moniteur pour détection des changements de fichiers et fuites mémoire
    'monitor' => [
        'handler' => app\process\Monitor::class, // Classe gestionnaire
        'reloadable' => false, // Ce processus n'exécute pas reload
        'constructor' => [ // Paramètres du constructeur du gestionnaire
            // Répertoires à surveiller, éviter trop nombreux (ralentit la détection)
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Extensions de fichiers à surveiller
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // Autres options
            'options' => [
                // Activer la surveillance de fichiers, Linux uniquement, désactivé par défaut en mode démon
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // Activer la surveillance mémoire, Linux uniquement
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// Retourner une instance du conteneur d'injection de dépendances PSR-11
return new Webman\Container;
```

#### dependence.php
```php
// Configurer les services et dépendances dans le conteneur d'injection de dépendances
return [];
```

#### route.php
```php

use support\Route;
// Définir la route pour le chemin /test
Route::any('/test', function (Request $request) {
    return response('test');
});
```

#### view.php
```php
use support\view\Raw;
use support\view\Twig;
use support\view\Blade;
use support\view\ThinkPHP;

return [
    'handler' => Raw::class // Classe gestionnaire de vues par défaut
];
```

### autoload.php
```php
// Configurer les fichiers d'autochargement du framework
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php',
        base_path() . '/support/Response.php',
    ]
];
```

#### cache.php
```php
// Configuration du cache
return [
    'default' => 'file', // Pilote par défaut
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // Emplacement du stockage du cache
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // Nom de connexion Redis, référence redis.php
        ],
        'array' => [
            'driver' => 'array' // Cache en mémoire, effacé au redémarrage
        ]
    ]
];
```

#### redis.php
```php
return [
    'default' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 0,
    ],
];
```

#### database.php
```php
return [
 // Base de données par défaut
 'default' => 'mysql',
 // Configurations de connexion
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
     ],

     'sqlite' => [
         'driver'   => 'sqlite',
         'database' => '',
         'prefix'   => '',
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
     ],
 ],
];
```

#### exception.php
```php
return [
    // Définir la classe gestionnaire d'exceptions
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // Gestionnaire
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // Nom du fichier log
                    7, //$maxFiles // Conserver les logs pendant 7 jours
                    Monolog\Logger::DEBUG, // Niveau de log
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // Formateur
                    'constructor' => [null, 'Y-m-d H:i:s', true], // Paramètres du formateur
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // Type
    'type' => 'file', // or redis or redis_cluster
     // Gestionnaire
    'handler' => FileSessionHandler::class,
     // Configuration
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // Répertoire de stockage
        ],
        'redis' => [
            'host' => '127.0.0.1',
            'port' => 6379,
            'auth' => '',
            'timeout' => 2,
            'database' => '',
            'prefix' => 'redis_session_',
        ],
        'redis_cluster' => [
            'host' => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth' => '',
            'prefix' => 'redis_session_',
        ]
    ],
    'session_name' => 'PHPSID', // Nom de session
    'auto_update_timestamp' => false, // Mise à jour auto du timestamp, éviter l'expiration
    'lifetime' => 7*24*60*60, // Durée de vie
    'cookie_lifetime' => 365*24*60*60, // Durée de vie du cookie
    'cookie_path' => '/', // Chemin du cookie
    'domain' => '', // Domaine du cookie
    'http_only' => true, // HTTP uniquement
    'secure' => false, // HTTPS uniquement
    'same_site' => '', // Attribut SameSite
    'gc_probability' => [1, 1000], // Probabilité de garbage collection des sessions
];
```

#### middleware.php
```php
// Configurer les middlewares
return [];
```

#### static.php
```php
return [
    'enable' => true, // Activer le service de fichiers statiques webman
    'middleware' => [ // Middleware des fichiers statiques, pour politique de cache, CORS etc.
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // Langue par défaut
    'locale' => 'zh_CN',
    // Langues de secours
    'fallback_locale' => ['zh_CN', 'en'],
    // Emplacement des fichiers de traduction
    'path' => base_path() . '/resource/translations',
];
```
