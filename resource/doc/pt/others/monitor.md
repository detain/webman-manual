# Monitoração de processos
O webman inclui um processo de monitoração integrado que suporta duas funções:
1. Monitora atualizações de arquivos e recarrega automaticamente o novo código de negócios (geralmente usado no desenvolvimento).
2. Monitora o uso de memória de todos os processos; se um processo estiver prestes a exceder o limite `memory_limit` no `php.ini`, será reiniciado automaticamente de forma segura (sem impacto no negócio).

## Configuração de monitoração
Configuração de `monitor` em `config/process.php`:
```php

global $argv;

return [
    // Detecção de atualização de arquivos e recarregamento automático
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // Monitorar esses diretórios
            'monitorDir' => array_merge([    // Quais diretórios precisam ser monitorados
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Arquivos com essas extensões serão monitorados
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // Habilitar monitoração de arquivos
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // Habilitar monitoração de memória
            ]
        ]
    ]
];
```
`monitorDir` configura quais diretórios monitorar para atualizações (não é aconselhável ter muitos arquivos nos diretórios monitorados).
`monitorExtensions` configura quais extensões de arquivo monitorar nos diretórios `monitorDir`.
Quando `options.enable_file_monitor` é `true`, a monitoração de atualizações de arquivos é ativada (por padrão no Linux em modo debug).
Quando `options.enable_memory_monitor` é `true`, a monitoração de memória é ativada (não suportada no Windows).

> **Dica**
> No Windows, a monitoração de atualizações de arquivos só é ativada ao executar `windows.bat` ou `php windows.php`.
