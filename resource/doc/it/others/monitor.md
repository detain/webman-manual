# Monitoraggio dei processi
webman include un processo di monitoraggio integrato che supporta due funzionalità:
1. Monitora gli aggiornamenti dei file e ricarica automaticamente il nuovo codice di business (generalmente usato durante lo sviluppo).
2. Monitora l'utilizzo della memoria di tutti i processi; se un processo sta per superare il limite `memory_limit` in `php.ini`, viene riavviato automaticamente in modo sicuro (senza impatto sul business).

## Configurazione del monitoraggio
Configurazione di `monitor` in `config/process.php`:
```php

global $argv;

return [
    // Rilevamento aggiornamenti file e ricarica automatica
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // Monitorare queste directory
            'monitorDir' => array_merge([    // Quali directory devono essere monitorate
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // I file con queste estensioni saranno monitorati
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // Abilitare il monitoraggio dei file
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // Abilitare il monitoraggio della memoria
            ]
        ]
    ]
];
```
`monitorDir` configura quali directory monitorare per gli aggiornamenti (non è consigliabile avere troppi file nelle directory monitorate).
`monitorExtensions` configura quali estensioni di file monitorare nelle directory `monitorDir`.
Quando `options.enable_file_monitor` è `true`, si abilita il monitoraggio degli aggiornamenti dei file (di default su Linux in modalità debug).
Quando `options.enable_memory_monitor` è `true`, si abilita il monitoraggio della memoria (non supportato su Windows).

> **Suggerimento**
> Su Windows, il monitoraggio degli aggiornamenti dei file è abilitato solo eseguendo `windows.bat` o `php windows.php`.
