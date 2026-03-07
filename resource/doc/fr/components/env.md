# Composant ENV vlucas/phpdotenv

## Description
`vlucas/phpdotenv` est un composant de chargement de variables d'environnement, utilisé pour différencier la configuration selon les environnements (développement, tests, etc.).

## Adresse du projet

https://github.com/vlucas/phpdotenv
  
## Installation
 
```php
composer require vlucas/phpdotenv
 ```
  
## Utilisation

### Créer un fichier `.env` à la racine du projet
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### Modifier le fichier de configuration
**config/database.php**
```php
return [
    // Base de données par défaut
    'default' => 'mysql',

    // Configurations des différentes bases de données
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **Conseil**
> Il est recommandé d'ajouter le fichier `.env` à la liste `.gitignore` pour éviter de le committer. Ajoutez un fichier exemple `.env.example` au dépôt. Lors du déploiement, copiez `.env.example` en `.env` et adaptez la configuration selon l'environnement. Ainsi, le projet chargera des configurations différentes selon l'environnement.

> **Attention**
> `vlucas/phpdotenv` peut présenter des bugs avec PHP en version TS (Thread Safe). Utilisez la version NTS (Non-Thread-Safe). Pour connaître la version de PHP installée, exécutez `php -v`.

## Plus d'informations

Visitez https://github.com/vlucas/phpdotenv
  
