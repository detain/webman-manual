# Registo

O webman utiliza o [monolog/monolog](https://github.com/Seldaek/monolog) para gerir registos.

## Utilização
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

## Métodos disponibilizados
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

## Configuração
```php
return [
    // Canal de registo predefinido
    'default' => [
        // Manipuladores do canal predefinido, pode configurar vários
        'handlers' => [
            [   
                // Nome da classe do manipulador
                'class' => Monolog\Handler\RotatingFileHandler::class,
                // Parâmetros do construtor da classe do manipulador
                'constructor' => [
                    runtime_path() . '/logs/webman.log',
                    Monolog\Logger::DEBUG,
                ],
                // Formatação relacionada
                'formatter' => [
                    // Nome da classe do processador de formatação
                    'class' => Monolog\Formatter\LineFormatter::class,
                    // Parâmetros do construtor da classe do processador de formatação
                    'constructor' => [ null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
];
```

## Múltiplos Canais
O monolog suporta múltiplos canais, sendo `default` o canal predefinido. Se desejar adicionar um canal `log2`, a configuração será semelhante à seguinte:
```php
return [
    // Canal de registo predefinido
    'default' => [
        // Manipuladores do canal predefinido, pode configurar vários
        'handlers' => [
            [   
                // Nome da classe do manipulador
                'class' => Monolog\Handler\RotatingFileHandler::class,
                // Parâmetros do construtor da classe do manipulador
                'constructor' => [
                    runtime_path() . '/logs/webman.log',
                    Monolog\Logger::DEBUG,
                ],
                // Formatação relacionada
                'formatter' => [
                    // Nome da classe do processador de formatação
                    'class' => Monolog\Formatter\LineFormatter::class,
                    // Parâmetros do construtor da classe do processador de formatação
                    'constructor' => [ null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
    // Canal log2
    'log2' => [
        // Manipuladores do canal log2, pode configurar vários
        'handlers' => [
            [   
                // Nome da classe do manipulador
                'class' => Monolog\Handler\RotatingFileHandler::class,
                // Parâmetros do construtor da classe do manipulador
                'constructor' => [
                    runtime_path() . '/logs/log2.log',
                    Monolog\Logger::DEBUG,
                ],
                // Formatação relacionada
                'formatter' => [
                    // Nome da classe do processador de formatação
                    'class' => Monolog\Formatter\LineFormatter::class,
                    // Parâmetros do construtor da classe do processador de formatação
                    'constructor' => [ null, 'Y-m-d H:i:s', true],
                ],
            ]
        ],
    ],
];
```

Para utilizar o canal `log2`, a utilização é a seguinte:
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
