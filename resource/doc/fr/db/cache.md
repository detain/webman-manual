# Cache

[webman/cache](https://github.com/webman-php/cache) est un composant de cache basé sur [symfony/cache](https://github.com/symfony/cache), compatible avec les environnements coroutine et non coroutine, et prenant en charge le pool de connexions.

## Installation

```php
composer require -W webman/cache
```

## Exemple
```php
<?php
namespace app\controller;

use support\Request;
use support\Cache;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Cache::set($key, rand());
        return response(Cache::get($key));
    }
}
```

## Emplacement du fichier de configuration
Le fichier de configuration se trouve dans `config/cache.php`. Créez-le manuellement s'il n'existe pas.

## Contenu du fichier de configuration
```php
<?php
return [
    'default' => 'file',
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default'
        ],
        'array' => [
            'driver' => 'array'
        ],
        'apcu' => [
            'driver' => 'apcu'
        ]
    ]
];
```
`stores.driver` prend en charge quatre pilotes : **file**, **redis**, **array** et **apcu**.

#### Pilote file
Pilote par défaut. Aucune dépendance externe. Permet le partage du cache entre processus. Ne permet pas le partage entre plusieurs serveurs.

#### Pilote array
Stockage en mémoire offrant les meilleures performances mais consommant la mémoire. Ne permet pas le partage entre processus ni entre serveurs. Les données sont perdues au redémarrage du processus. S’utilise généralement pour des projets avec peu de données en cache.

#### Pilote apcu
Stockage en mémoire. Les performances ne sont dépassées que par array. Permet le partage du cache entre processus. Ne permet pas le partage entre plusieurs serveurs. Les données sont perdues au redémarrage du processus. S’utilise généralement pour des projets avec peu de données en cache.

> Nécessite l’installation et l’activation de l’[extension APCu](https://pecl.php.net/package/APCu). Non recommandé pour des écritures/suppressions fréquentes du cache, car cela peut dégrader nettement les performances.

#### Pilote redis
Dépend du composant [webman/redis](./redis.md). Permet le partage du cache entre processus et serveurs.

**stores.redis.connection**

`stores.redis.connection` correspond à la clé définie dans `config/redis.php`. Avec Redis, la configuration de `webman/redis` est réutilisée, y compris le pool de connexions.

**Il est recommandé d’ajouter une configuration Redis dédiée au cache dans `config/redis.php`, par exemple :**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== À ajouter
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

Puis définir `stores.redis.connection` sur `cache`. Le fichier `config/cache.php` final peut ressembler à ceci :

```php
<?php
return [
    'default' => 'redis', // <====
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache' // <====
        ],
        'array' => [
            'driver' => 'array'
        ]
    ]
];
```

## Changer de stockage
Vous pouvez changer manuellement le stockage pour utiliser différents pilotes, par exemple :

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **Conseil**
> Les noms des clés de cache sont restreints par [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) et ne doivent pas contenir l’un des caractères suivants : `{}()/\@:`. Depuis `symfony/cache` 7.2.4, cette vérification peut être contournée via l’option PHP ini `zend.assertions=-1`.

## Utiliser d’autres composants de cache

Consultez [Other Databases](others.md#ThinkCache) pour le composant [ThinkCache](https://github.com/webman-php/think-cache).
