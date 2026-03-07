# Log

webman utilizza [monolog/monolog](https://github.com/Seldaek/monolog) per gestire i log.

## Utilizzo
```php
<?php
namespace app\controller;

use support\Request;
use support\Log;

class FooController
{
    public function index(Request $request)
    {
        Log::info('log test');
        return response('hello index');
    }
}
```

## Metodi forniti
```php
Log::log($level, $message, array $context = [])
Log::debug($message, array $context = [])
Log::info($message, array $context = [])
Log::notice($message, array $context = [])
Log::warning($message, array $context = [])
Log::error($message, array $context = [])
Log::critical($message, array $context = [])
Log::alert($message, array $context = [])
Log::emergency($message, array $context = [])
```
Equivalente a
```php
$log = Log::channel('default');
$log->log($level, $message, array $context = [])
$log->debug($message, array $context = [])
$log->info($message, array $context = [])
$log->notice($message, array $context = [])
$log->warning($message, array $context = [])
$log->error($message, array $context = [])
$log->critical($message, array $context = [])
$log->alert($message, array $context = [])
$log->emergency($message, array $context = [])
```

## Configurazione
```php
return [
    // Canale di log predefinito
    'default' => [
        // Gestori del canale predefinito, è possibile impostarne più di uno
        'handlers' => [
            [   
                // Nome della classe del gestore
                'class' => Monolog\Handler\RotatingFileHandler::class,
                // Parametri del costruttore della classe del gestore
                'constructor' => [
                    runtime_path() . '/logs/webman.log',
                    Monolog\Logger::DEBUG,
                ],
                // Formattazione correlata
                'formatter' => [
                    // Nome della classe del formattatore
                    'class' => Monolog\Formatter\LineFormatter::class,
                    // Parametri del costruttore della classe del formattatore
                    'constructor' => [ null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
];
```

## Canali multipli
Monolog supporta canali multipli, utilizza il canale `default` per impostazione predefinita. Se si vuole aggiungere un canale `log2`, la configurazione è simile a quanto segue:
```php
return [
    // Canale di log predefinito
    'default' => [
        // Gestori del canale predefinito, è possibile impostarne più di uno
        'handlers' => [
            [   
                // Nome della classe del gestore
                'class' => Monolog\Handler\RotatingFileHandler::class,
                // Parametri del costruttore della classe del gestore
                'constructor' => [
                    runtime_path() . '/logs/webman.log',
                    Monolog\Logger::DEBUG,
                ],
                // Formattazione correlata
                'formatter' => [
                    // Nome della classe del formattatore
                    'class' => Monolog\Formatter\LineFormatter::class,
                    // Parametri del costruttore della classe del formattatore
                    'constructor' => [ null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
    // Canale log2
    'log2' => [
        // Gestori del canale log2, è possibile impostarne più di uno
        'handlers' => [
            [   
                // Nome della classe del gestore
                'class' => Monolog\Handler\RotatingFileHandler::class,
                // Parametri del costruttore della classe del gestore
                'constructor' => [
                    runtime_path() . '/logs/log2.log',
                    Monolog\Logger::DEBUG,
                ],
                // Formattazione correlata
                'formatter' => [
                    // Nome della classe del formattatore
                    'class' => Monolog\Formatter\LineFormatter::class,
                    // Parametri del costruttore della classe del formattatore
                    'constructor' => [ null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
];
```

Quando si utilizza il canale `log2`, l'uso è il seguente:
```php
<?php
namespace app\controller;

use support\Request;
use support\Log;

class FooController
{
    public function index(Request $request)
    {
        $log = Log::channel('log2');
        $log->info('log2 test');
        return response('hello index');
    }
}
```
