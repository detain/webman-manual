# File di configurazione

## Posizione
I file di configurazione di webman si trovano nella directory `config/`. È possibile utilizzare la funzione `config()` per accedere alle configurazioni corrispondenti nel progetto.

## Accesso alle configurazioni

Ottenere tutte le configurazioni:
```php
config();
```

Ottenere tutte le configurazioni in `config/app.php`:
```php
config('app');
```

Ottenere la configurazione `debug` in `config/app.php`:
```php
config('app.debug');
```

Se la configurazione è un array, è possibile utilizzare `.` per accedere ai valori annidati. Ad esempio:
```php
config('file.key1.key2');
```

## Valore predefinito
```php
config($key, $default);
```
Passare il valore predefinito come secondo parametro. Se la configurazione non esiste, verrà restituito il valore predefinito. Se la configurazione non esiste e non è impostato alcun valore predefinito, viene restituito `null`.

## Configurazione personalizzata
Gli sviluppatori possono aggiungere i propri file di configurazione nella directory `config/`. Ad esempio:

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**Utilizzo nell'accesso alle configurazioni**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## Modifica delle configurazioni
Webman non supporta modifiche dinamiche della configurazione. Tutte le configurazioni devono essere modificate manualmente nei relativi file di configurazione, quindi ricaricare o riavviare l'applicazione.

> **Nota**
> La configurazione del server `config/server.php` e la configurazione dei processi `config/process.php` non supportano il reload. È necessario riavviare l'applicazione perché le modifiche abbiano effetto.

## Promemoria importante
Se si creano file di configurazione in una sottodirectory di `config/`, ad esempio `config/order/status.php`, è necessario un file `app.php` nella directory `config/order/` con il seguente contenuto:
```php
<?php
return [
    'enable' => true,
];
```
`enable` impostato a `true` indica al framework di caricare le configurazioni da questa directory.
La struttura della directory di configurazione dovrebbe essere la seguente:
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
È quindi possibile accedere all'array o alle chiavi specifiche di `status.php` tramite `config.order.status`.


## Riferimento ai file di configurazione

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // Porta di ascolto (rimosso dalla 1.6.0, configurato in config/process.php)
    'transport' => 'tcp', // Protocollo di trasporto (rimosso dalla 1.6.0, configurato in config/process.php)
    'context' => [], // SSL ecc. (rimosso dalla 1.6.0, configurato in config/process.php)
    'name' => 'webman', // Nome processo (rimosso dalla 1.6.0, configurato in config/process.php)
    'count' => cpu_count() * 4, // Numero processi (rimosso dalla 1.6.0, configurato in config/process.php)
    'user' => '', // Utente (rimosso dalla 1.6.0, configurato in config/process.php)
    'group' => '', // Gruppo (rimosso dalla 1.6.0, configurato in config/process.php)
    'reusePort' => false, // Abilita riuso porta (rimosso dalla 1.6.0, configurato in config/process.php)
    'event_loop' => '',  // Classe event loop, selezionata automaticamente per impostazione predefinita
    'stop_timeout' => 2, // Tempo massimo di attesa alla ricezione del segnale stop/restart/reload, uscita forzata se il processo non termina in tempo
    'pid_file' => runtime_path() . '/webman.pid', // Posizione file PID
    'status_file' => runtime_path() . '/webman.status', // Posizione file stato
    'stdout_file' => runtime_path() . '/logs/stdout.log', // Posizione file stdout, qui viene scritto tutto l'output dopo l'avvio di webman
    'log_file' => runtime_path() . '/logs/workerman.log', // Posizione file log Workerman
    'max_package_size' => 10 * 1024 * 1024 // Dimensione massima pacchetto, 10M. La dimensione di upload dei file è limitata da questo valore
];
```

#### app.php
```php
return [
    'debug' => true,  // Modalità debug, abilita stack trace ecc. sugli errori. Disabilitare in produzione
    'error_reporting' => E_ALL, // Livello di segnalazione errori
    'default_timezone' => 'Asia/Shanghai', // Fuso orario predefinito
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // Percorso directory pubblica
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // Percorso directory runtime
    'controller_suffix' => 'Controller', // Suffisso controller
    'controller_reuse' => false, // Se riutilizzare i controller
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // Configurazione processi webman
    'webman' => [ 
        'handler' => Http::class, // Classe gestore processo
        'listen' => 'http://0.0.0.0:8787', // Indirizzo di ascolto
        'count' => cpu_count() * 4, // Numero processi, 4x CPU per impostazione predefinita
        'user' => '', // Utente processo, utilizzare utente a bassi privilegi
        'group' => '', // Gruppo processo, utilizzare gruppo a bassi privilegi
        'reusePort' => false, // Abilita reusePort, distribuisce le connessioni tra i processi worker
        'eventLoop' => '', // Classe event loop, usa server.event_loop quando vuoto
        'context' => [], // Contesto ascolto, es. SSL
        'constructor' => [ // Parametri costruttore per il gestore processo, qui classe Http
            'requestClass' => Request::class, // Classe richiesta personalizzata
            'logger' => Log::channel('default'), // Istanza logger
            'appPath' => app_path(), // Percorso directory app
            'publicPath' => public_path() // Percorso directory pubblica
        ]
    ],
    // Processo monitor per auto-reload su cambio file e rilevamento perdite memoria
    'monitor' => [
        'handler' => app\process\Monitor::class, // Classe gestore
        'reloadable' => false, // Questo processo non esegue reload
        'constructor' => [ // Parametri costruttore per il gestore processo
            // Directory da monitorare, evitare troppe perché rallentano il rilevamento
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Estensioni file da monitorare per modifiche
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // Altre opzioni
            'options' => [
                // Abilita monitoraggio file, solo Linux, disabilitato per impostazione predefinita in modalità daemon
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // Abilita monitoraggio memoria, solo Linux
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// Restituisce un'istanza del contenitore di iniezione dipendenze PSR-11
return new Webman\Container;
```

#### dependence.php
```php
// Configura servizi e dipendenze nel contenitore di iniezione dipendenze
return [];
```

#### route.php
```php

use support\Route;
// Definisce rotta per il percorso /test
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
    'handler' => Raw::class // Classe gestore view predefinita
];
```

### autoload.php
```php
// Configura i file di autoload del framework
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
// Configurazione cache
return [
    'default' => 'file', // Driver predefinito
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // Percorso archiviazione file cache
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // Nome connessione Redis, si riferisce alla config in redis.php
        ],
        'array' => [
            'driver' => 'array' // Cache in memoria, cancellata al riavvio
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
 // Database predefinito
 'default' => 'mysql',
 // Configurazioni connessioni database
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
    // Imposta la classe gestore eccezioni
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // Gestore
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // Nome file log
                    7, //$maxFiles // Mantenere log per 7 giorni
                    Monolog\Logger::DEBUG, // Livello log
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // Formatter
                    'constructor' => [null, 'Y-m-d H:i:s', true], // Parametri formatter
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // Tipo
    'type' => 'file', // oppure redis o redis_cluster
     // Gestore
    'handler' => FileSessionHandler::class,
     // Configurazione
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // Directory archiviazione
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
    'session_name' => 'PHPSID', // Nome sessione
    'auto_update_timestamp' => false, // Aggiorna automaticamente timestamp per prevenire scadenza sessione
    'lifetime' => 7*24*60*60, // Durata
    'cookie_lifetime' => 365*24*60*60, // Durata cookie
    'cookie_path' => '/', // Percorso cookie
    'domain' => '', // Dominio cookie
    'http_only' => true, // Solo HTTP
    'secure' => false, // Solo HTTPS
    'same_site' => '', // Attributo SameSite
    'gc_probability' => [1, 1000], // Probabilità garbage collection sessione
];
```

#### middleware.php
```php
// Configura middleware
return [];
```

#### static.php
```php
return [
    'enable' => true, // Abilita servizio file statici webman
    'middleware' => [ // Middleware file statici, per politica cache, CORS ecc.
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // Lingua predefinita
    'locale' => 'zh_CN',
    // Lingue di fallback
    'fallback_locale' => ['zh_CN', 'en'],
    // Posizione file traduzioni
    'path' => base_path() . '/resource/translations',
];
```
