# Cache

[webman/cache](https://github.com/webman-php/cache) è un componente di cache basato su [symfony/cache](https://github.com/symfony/cache), compatibile con ambienti coroutine e non coroutine, con supporto per il connection pool.

## Installazione

```php
composer require -W webman/cache
```

## Esempio
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

## Posizione del file di configurazione
Il file di configurazione si trova in `config/cache.php`. Crearlo manualmente se non esiste.

## Contenuto del file di configurazione
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
`stores.driver` supporta quattro driver: **file**, **redis**, **array** e **apcu**.

#### Driver file
Driver predefinito. Nessuna dipendenza esterna. Consente la condivisione della cache tra processi. Non consente la condivisione tra più server.

#### Driver array
Archiviazione in memoria con le migliori prestazioni, ma consuma memoria. Non consente la condivisione tra processi o server. I dati si perdono al riavvio del processo. Tipico per progetti con volume di cache ridotto.

#### Driver apcu
Archiviazione in memoria. Le prestazioni sono seconde solo a array. Consente la condivisione della cache tra processi. Non consente la condivisione tra più server. I dati si perdono al riavvio del processo. Tipico per progetti con volume di cache ridotto.

> Richiede l'installazione e l'attivazione dell'[estensione APCu](https://pecl.php.net/package/APCu). Non consigliato per scenari con frequenti scritture/cancellazioni della cache, poiché può causare un degrado significativo delle prestazioni.

#### Driver redis
Dipende dal componente [webman/redis](./redis.md). Consente la condivisione della cache tra processi e server.

**stores.redis.connection**

`stores.redis.connection` corrisponde alla chiave definita in `config/redis.php`. Quando si usa Redis, riutilizza la configurazione di `webman/redis`, inclusi i parametri del connection pool.

**Si consiglia di aggiungere una configurazione Redis dedicata al cache in `config/redis.php`, ad esempio:**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== Aggiungere nuovo
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

Poi impostare `stores.redis.connection` su `cache`. Il file `config/cache.php` finale dovrebbe essere simile a:

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

## Cambio di storage
È possibile passare manualmente a un altro storage per usare driver diversi, ad esempio:

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **Suggerimento**
> I nomi delle chiavi cache sono vincolati da [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) e non devono contenere nessuno di questi caratteri: `{}()/\@:`. A partire da `symfony/cache` 7.2.4, questo controllo può essere aggirato con l'opzione PHP ini `zend.assertions=-1`.

## Utilizzo di altri componenti cache

Consultare [Altri database](others.md#ThinkCache) per il componente [ThinkCache](https://github.com/webman-php/think-cache).
