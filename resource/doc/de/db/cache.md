# Cache

[webman/cache](https://github.com/webman-php/cache) ist eine auf [symfony/cache](https://github.com/symfony/cache) basierende Cache-Komponente, kompatibel mit Coroutine- und Nicht-Coroutine-Umgebungen und mit Verbindungspool-Unterstützung.

## Installation

```php
composer require -W webman/cache
```

## Beispiel
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

## Speicherort der Konfigurationsdatei
Die Konfigurationsdatei befindet sich unter `config/cache.php`. Erstellen Sie sie bei Bedarf manuell.

## Inhalt der Konfigurationsdatei
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
`stores.driver` unterstützt vier Treiber: **file**, **redis**, **array** und **apcu**.

#### file-Treiber
Der Standard-Treiber. Keine externen Abhängigkeiten. Unterstützt die gemeinsame Nutzung von Cache über Prozesse hinweg. Unterstützt keine gemeinsame Nutzung über mehrere Server.

#### array-Treiber
Speicherung im Arbeitsspeicher mit bester Leistung, aber mit Speicherverbrauch. Keine gemeinsame Nutzung über Prozesse oder Server. Daten gehen beim Neustart des Prozesses verloren. Typisch für Projekte mit geringem Cache-Volumen.

#### apcu-Treiber
Speicherung im Arbeitsspeicher. Leistung nur von array übertroffen. Unterstützt gemeinsame Cache-Nutzung über Prozesse. Unterstützt keine gemeinsame Nutzung über mehrere Server. Daten gehen beim Neustart des Prozesses verloren. Typisch für Projekte mit geringem Cache-Volumen.

> Erfordert die Installation und Aktivierung der [APCu-Erweiterung](https://pecl.php.net/package/APCu). Nicht empfohlen bei häufigem Schreiben/Löschen von Cache, da dies die Leistung deutlich beeinträchtigen kann.

#### redis-Treiber
Abhängig von der Komponente [webman/redis](./redis.md). Unterstützt gemeinsame Cache-Nutzung über Prozesse und Server.

**stores.redis.connection**

`stores.redis.connection` entspricht dem Schlüssel in `config/redis.php`. Bei Nutzung von Redis wird die Konfiguration von `webman/redis` inkl. Verbindungspool übernommen.

**Es wird empfohlen, in `config/redis.php` eine eigene Redis-Konfiguration für Cache anzulegen, z. B.:**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== Neu hinzufügen
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

Setzen Sie dann `stores.redis.connection` auf `cache`. Die finale `config/cache.php` könnte so aussehen:

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

## Speicher wechseln
Sie können manuell den Speicher wechseln, um andere Treiber zu nutzen, z. B.:

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **Hinweis**
> Cache-Schlüsselnamen unterliegen [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) und dürfen keines dieser Zeichen enthalten: `{}()/\@:`. Seit `symfony/cache` 7.2.4 kann diese Prüfung durch die PHP-ini-Option `zend.assertions=-1` umgangen werden.

## Andere Cache-Komponenten verwenden

Siehe [Andere Datenbanken](others.md#ThinkCache) für die [ThinkCache](https://github.com/webman-php/think-cache)-Komponente.
