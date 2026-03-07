# Arquivos de configuração

## Localização
Os arquivos de configuração do webman estão localizados no diretório `config/`. Você pode usar a função `config()` para acessar as configurações correspondentes no seu projeto.

## Acessando configurações

Obter todas as configurações:
```php
config();
```

Obter todas as configurações em `config/app.php`:
```php
config('app');
```

Obter a configuração `debug` em `config/app.php`:
```php
config('app.debug');
```

Se a configuração for um array, você pode usar `.` para acessar valores aninhados. Por exemplo:
```php
config('file.key1.key2');
```

## Valor padrão
```php
config($key, $default);
```
Passe o valor padrão como segundo parâmetro. Se a configuração não existir, o valor padrão será retornado. Se a configuração não existir e nenhum padrão for definido, `null` será retornado.

## Configuração personalizada
Os desenvolvedores podem adicionar seus próprios arquivos de configuração no diretório `config/`. Por exemplo:

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**Uso ao acessar configurações**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## Modificando configurações
O Webman não suporta alterações dinâmicas de configuração. Todas as configurações devem ser modificadas manualmente nos arquivos de configuração correspondentes e, em seguida, recarregar ou reiniciar a aplicação.

> **Nota**
> A configuração do servidor `config/server.php` e a configuração de processos `config/process.php` não suportam reload. É necessário reiniciar a aplicação para que as alterações tenham efeito.

## Lembrete importante
Se você criar arquivos de configuração em um subdiretório dentro de `config/`, por exemplo `config/order/status.php`, será necessário um arquivo `app.php` no diretório `config/order/` com o seguinte conteúdo:
```php
<?php
return [
    'enable' => true,
];
```
`enable` definido como `true` informa ao framework para carregar as configurações deste diretório.
A estrutura do diretório de configuração deve ser assim:
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
Você pode então acessar o array ou chaves específicas de `status.php` via `config.order.status`.


## Referência dos arquivos de configuração

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // Porta de escuta (removido desde 1.6.0, configurado em config/process.php)
    'transport' => 'tcp', // Protocolo de transporte (removido desde 1.6.0, configurado em config/process.php)
    'context' => [], // SSL etc. (removido desde 1.6.0, configurado em config/process.php)
    'name' => 'webman', // Nome do processo (removido desde 1.6.0, configurado em config/process.php)
    'count' => cpu_count() * 4, // Número de processos (removido desde 1.6.0, configurado em config/process.php)
    'user' => '', // Usuário (removido desde 1.6.0, configurado em config/process.php)
    'group' => '', // Grupo (removido desde 1.6.0, configurado em config/process.php)
    'reusePort' => false, // Habilitar reutilização de porta (removido desde 1.6.0, configurado em config/process.php)
    'event_loop' => '',  // Classe do event loop, selecionada automaticamente por padrão
    'stop_timeout' => 2, // Tempo máximo de espera ao receber sinal stop/restart/reload, saída forçada se o processo não terminar a tempo
    'pid_file' => runtime_path() . '/webman.pid', // Localização do arquivo PID
    'status_file' => runtime_path() . '/webman.status', // Localização do arquivo de status
    'stdout_file' => runtime_path() . '/logs/stdout.log', // Localização do arquivo stdout, toda saída após o webman iniciar é escrita aqui
    'log_file' => runtime_path() . '/logs/workerman.log', // Localização do arquivo de log Workerman
    'max_package_size' => 10 * 1024 * 1024 // Tamanho máximo do pacote, 10M. O tamanho de upload de arquivos é limitado por isso
];
```

#### app.php
```php
return [
    'debug' => true,  // Modo debug, habilita stack trace etc. em erros. Deve ser desabilitado em produção
    'error_reporting' => E_ALL, // Nível de relatório de erros
    'default_timezone' => 'Asia/Shanghai', // Fuso horário padrão
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // Caminho do diretório público
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // Caminho do diretório de runtime
    'controller_suffix' => 'Controller', // Sufixo do controller
    'controller_reuse' => false, // Se deve reutilizar controllers
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // Configuração de processos webman
    'webman' => [ 
        'handler' => Http::class, // Classe handler do processo
        'listen' => 'http://0.0.0.0:8787', // Endereço de escuta
        'count' => cpu_count() * 4, // Número de processos, 4x CPU por padrão
        'user' => '', // Usuário do processo, deve usar usuário de baixo privilégio
        'group' => '', // Grupo do processo, deve usar grupo de baixo privilégio
        'reusePort' => false, // Habilitar reusePort, distribui conexões entre os processos worker
        'eventLoop' => '', // Classe do event loop, usa server.event_loop quando vazio
        'context' => [], // Contexto de escuta, ex.: SSL
        'constructor' => [ // Parâmetros do construtor para o handler do processo, aqui a classe Http
            'requestClass' => Request::class, // Classe de requisição personalizada
            'logger' => Log::channel('default'), // Instância do logger
            'appPath' => app_path(), // Caminho do diretório app
            'publicPath' => public_path() // Caminho do diretório público
        ]
    ],
    // Processo monitor para auto-reload em alteração de arquivo e detecção de vazamento de memória
    'monitor' => [
        'handler' => app\process\Monitor::class, // Classe handler
        'reloadable' => false, // Este processo não executa reload
        'constructor' => [ // Parâmetros do construtor para o handler do processo
            // Diretórios a monitorar, evite muitos pois desacelera a detecção
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Extensões de arquivo a monitorar para alterações
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // Outras opções
            'options' => [
                // Habilitar monitoramento de arquivo, apenas Linux, desabilitado por padrão em modo daemon
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // Habilitar monitoramento de memória, apenas Linux
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// Retorna uma instância do container de injeção de dependência PSR-11
return new Webman\Container;
```

#### dependence.php
```php
// Configurar serviços e dependências no container de injeção de dependência
return [];
```

#### route.php
```php

use support\Route;
// Define rota para o caminho /test
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
    'handler' => Raw::class // Classe handler de view padrão
];
```

### autoload.php
```php
// Configurar arquivos de autoload do framework
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
// Configuração de cache
return [
    'default' => 'file', // Driver padrão
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // Caminho de armazenamento dos arquivos de cache
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // Nome da conexão Redis, refere-se à config em redis.php
        ],
        'array' => [
            'driver' => 'array' // Cache em memória, limpa no reinício
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
 // Banco de dados padrão
 'default' => 'mysql',
 // Configurações de conexão do banco de dados
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
    // Definir classe handler de exceção
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
                    runtime_path() . '/logs/webman.log', // Nome do arquivo de log
                    7, //$maxFiles // Manter logs por 7 dias
                    Monolog\Logger::DEBUG, // Nível de log
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // Formatter
                    'constructor' => [null, 'Y-m-d H:i:s', true], // Parâmetros do formatter
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
    'type' => 'file', // ou redis ou redis_cluster
     // Handler
    'handler' => FileSessionHandler::class,
     // Configuração
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // Diretório de armazenamento
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
    'session_name' => 'PHPSID', // Nome da sessão
    'auto_update_timestamp' => false, // Atualizar timestamp automaticamente para evitar expiração da sessão
    'lifetime' => 7*24*60*60, // Tempo de vida
    'cookie_lifetime' => 365*24*60*60, // Tempo de vida do cookie
    'cookie_path' => '/', // Caminho do cookie
    'domain' => '', // Domínio do cookie
    'http_only' => true, // Apenas HTTP
    'secure' => false, // Apenas HTTPS
    'same_site' => '', // Atributo SameSite
    'gc_probability' => [1, 1000], // Probabilidade de coleta de lixo da sessão
];
```

#### middleware.php
```php
// Configurar middleware
return [];
```

#### static.php
```php
return [
    'enable' => true, // Habilitar serviço de arquivos estáticos do webman
    'middleware' => [ // Middleware de arquivos estáticos, para política de cache, CORS etc.
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // Idioma padrão
    'locale' => 'zh_CN',
    // Idiomas de fallback
    'fallback_locale' => ['zh_CN', 'en'],
    // Localização dos arquivos de tradução
    'path' => base_path() . '/resource/translations',
];
```
