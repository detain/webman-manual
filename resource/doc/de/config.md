# Konfigurationsdatei

## Position
Die Konfigurationsdateien von webman befinden sich im Verzeichnis `config/`. Im Projekt können Sie die Funktion `config()` verwenden, um die entsprechenden Konfigurationen abzurufen.

## Abrufen von Konfigurationen

Alle Konfigurationen abrufen:
```php
config();
```

Alle Konfigurationen aus `config/app.php` abrufen:
```php
config('app');
```

Die `debug`-Konfiguration aus `config/app.php` abrufen:
```php
config('app.debug');
```

Wenn die Konfiguration ein Array ist, können Sie mit `.` verschachtelte Werte abrufen. Zum Beispiel:
```php
config('file.key1.key2');
```

## Standardwert
```php
config($key, $default);
```
Übergeben Sie den Standardwert als zweiten Parameter. Wenn die Konfiguration nicht existiert, wird der Standardwert zurückgegeben. Existiert die Konfiguration nicht und ist kein Standardwert gesetzt, wird `null` zurückgegeben.

## Benutzerdefinierte Konfiguration
Entwickler können eigene Konfigurationsdateien im Verzeichnis `config/` hinzufügen. Zum Beispiel:

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**Verwendung beim Abrufen der Konfiguration**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## Konfiguration ändern
Webman unterstützt keine dynamische Änderung der Konfiguration. Alle Konfigurationen müssen manuell in den entsprechenden Konfigurationsdateien geändert und die Anwendung dann per Reload oder Neustart aktualisiert werden.

> **Hinweis**
> Die Serverkonfiguration `config/server.php` und die Prozesskonfiguration `config/process.php` unterstützen kein Reload. Ein Neustart ist erforderlich, damit Änderungen wirksam werden.

## Wichtiger Hinweis
Wenn Sie Konfigurationsdateien in einem Unterverzeichnis von `config/` erstellen, z.B. `config/order/status.php`, benötigen Sie eine `app.php`-Datei im Verzeichnis `config/order/` mit folgendem Inhalt:
```php
<?php
return [
    'enable' => true,
];
```
`enable` auf `true` gesetzt bedeutet, dass das Framework Konfigurationen aus diesem Verzeichnis lädt.
Die Verzeichnisstruktur sollte so aussehen:
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
Sie können dann über `config.order.status` das Array oder bestimmte Schlüssel aus `status.php` abrufen.


## Konfigurationsdatei-Referenz

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // Listen-Port (entfernt seit 1.6.0, in config/process.php konfiguriert)
    'transport' => 'tcp', // Transportprotokoll (entfernt seit 1.6.0, in config/process.php konfiguriert)
    'context' => [], // SSL etc. (entfernt seit 1.6.0, in config/process.php konfiguriert)
    'name' => 'webman', // Prozessname (entfernt seit 1.6.0, in config/process.php konfiguriert)
    'count' => cpu_count() * 4, // Prozessanzahl (entfernt seit 1.6.0, in config/process.php konfiguriert)
    'user' => '', // Benutzer (entfernt seit 1.6.0, in config/process.php konfiguriert)
    'group' => '', // Gruppe (entfernt seit 1.6.0, in config/process.php konfiguriert)
    'reusePort' => false, // Port-Wiederverwendung (entfernt seit 1.6.0, in config/process.php konfiguriert)
    'event_loop' => '',  // Event-Loop-Klasse, Standardmäßig automatisch ausgewählt
    'stop_timeout' => 2, // Maximale Wartezeit bei stop/restart/reload-Signal, danach erzwungener Exit
    'pid_file' => runtime_path() . '/webman.pid', // PID-Datei-Speicherort
    'status_file' => runtime_path() . '/webman.status', // Status-Datei-Speicherort
    'stdout_file' => runtime_path() . '/logs/stdout.log', // stdout-Datei, alle Ausgaben nach webman-Start
    'log_file' => runtime_path() . '/logs/workerman.log', // Workerman-Log-Datei-Speicherort
    'max_package_size' => 10 * 1024 * 1024 // Maximale Paketgröße, 10M. Begrenzt Upload-Dateigröße
];
```

#### app.php
```php
return [
    'debug' => true,  // Debug-Modus, ermöglicht Stack-Trace bei Fehlern. In Produktion deaktivieren
    'error_reporting' => E_ALL, // Fehlerberichtsstufe
    'default_timezone' => 'Asia/Shanghai', // Standardzeitzone
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // Public-Verzeichnispfad
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // Runtime-Verzeichnispfad
    'controller_suffix' => 'Controller', // Controller-Suffix
    'controller_reuse' => false, // Controller-Wiederverwendung
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // webman-Prozesskonfiguration
    'webman' => [ 
        'handler' => Http::class, // Prozess-Handler-Klasse
        'listen' => 'http://0.0.0.0:8787', // Listen-Adresse
        'count' => cpu_count() * 4, // Prozessanzahl, Standard 4x CPU
        'user' => '', // Prozessbenutzer, niedrig-privilegierter Benutzer verwenden
        'group' => '', // Prozessgruppe, niedrig-privilegierte Gruppe verwenden
        'reusePort' => false, // reusePort aktivieren, verteilt Verbindungen auf Worker-Prozesse
        'eventLoop' => '', // Event-Loop-Klasse, verwendet server.event_loop wenn leer
        'context' => [], // Listen-Kontext, z.B. SSL
        'constructor' => [ // Konstruktorparameter für Prozess-Handler, hier Http-Klasse
            'requestClass' => Request::class, // Benutzerdefinierte Request-Klasse
            'logger' => Log::channel('default'), // Logger-Instanz
            'appPath' => app_path(), // App-Verzeichnispfad
            'publicPath' => public_path() // Public-Verzeichnispfad
        ]
    ],
    // Monitor-Prozess für Dateiänderungs-Erkennung und Speicherleck-Erkennung
    'monitor' => [
        'handler' => app\process\Monitor::class, // Handler-Klasse
        'reloadable' => false, // Dieser Prozess führt kein Reload aus
        'constructor' => [ // Konstruktorparameter für Prozess-Handler
            // Zu überwachende Verzeichnisse, nicht zu viele (verlangsamt Erkennung)
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Dateierweiterungen für Änderungsüberwachung
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // Weitere Optionen
            'options' => [
                // Dateiüberwachung aktivieren, nur Linux, standardmäßig im Daemon-Modus deaktiviert
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // Speicherüberwachung aktivieren, nur Linux
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// PSR-11 Dependency-Injection-Container-Instanz zurückgeben
return new Webman\Container;
```

#### dependence.php
```php
// Dienste und Abhängigkeiten im DI-Container konfigurieren
return [];
```

#### route.php
```php

use support\Route;
// Route für /test-Pfad definieren
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
    'handler' => Raw::class // Standard-View-Handler-Klasse
];
```

### autoload.php
```php
// Framework-Autoload-Dateien konfigurieren
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
// Cache-Konfiguration
return [
    'default' => 'file', // Standard-Treiber
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // Cache-Datei-Speicherort
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // Redis-Verbindungsname, bezieht sich auf redis.php
        ],
        'array' => [
            'driver' => 'array' // Speicher-Cache, bei Neustart geleert
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
 // Standard-Datenbank
 'default' => 'mysql',
 // Datenbankverbindungs-Konfigurationen
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
    // Exception-Handler-Klasse festlegen
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // Handler
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // Log-Dateiname
                    7, //$maxFiles // Logs 7 Tage behalten
                    Monolog\Logger::DEBUG, // Log-Stufe
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // Formatierer
                    'constructor' => [null, 'Y-m-d H:i:s', true], // Formatierer-Parameter
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // Typ
    'type' => 'file', // or redis or redis_cluster
     // Handler
    'handler' => FileSessionHandler::class,
     // Konfiguration
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // Speicherverzeichnis
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
    'session_name' => 'PHPSID', // Sitzungsname
    'auto_update_timestamp' => false, // Zeitstempel automatisch aktualisieren, Sitzungsablauf verhindern
    'lifetime' => 7*24*60*60, // Lebensdauer
    'cookie_lifetime' => 365*24*60*60, // Cookie-Lebensdauer
    'cookie_path' => '/', // Cookie-Pfad
    'domain' => '', // Cookie-Domain
    'http_only' => true, // Nur HTTP
    'secure' => false, // Nur HTTPS
    'same_site' => '', // SameSite-Attribut
    'gc_probability' => [1, 1000], // Sitzungs-Garbage-Collection-Wahrscheinlichkeit
];
```

#### middleware.php
```php
// Middleware konfigurieren
return [];
```

#### static.php
```php
return [
    'enable' => true, // webman statische Dateibereitstellung aktivieren
    'middleware' => [ // Statische Datei-Middleware, für Cache-Richtlinie, CORS etc.
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // Standardsprache
    'locale' => 'zh_CN',
    // Fallback-Sprachen
    'fallback_locale' => ['zh_CN', 'en'],
    // Übersetzungsdatei-Speicherort
    'path' => base_path() . '/resource/translations',
];
```
