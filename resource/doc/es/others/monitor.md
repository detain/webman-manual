# Monitorización de procesos
webman incluye un proceso monitor integrado que soporta dos funciones:
1. Monitoriza las actualizaciones de archivos y recarga automáticamente el nuevo código de negocio (generalmente usado en desarrollo).
2. Monitoriza el uso de memoria de todos los procesos; si algún proceso está a punto de superar el límite `memory_limit` de `php.ini`, lo reinicia automáticamente de forma segura (sin afectar al negocio).

## Configuración de monitorización
Configuración de `monitor` en `config/process.php`:
```php

global $argv;

return [
    // Detección de actualización de archivos y recarga automática
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // Monitorizar estos directorios
            'monitorDir' => array_merge([    // Qué directorios deben ser monitorizados
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Se monitorizarán los archivos con estas extensiones
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // Activar monitorización de archivos
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // Activar monitorización de memoria
            ]
        ]
    ]
];
```
`monitorDir` configura qué directorios se monitorizan en busca de actualizaciones (no conviene que haya demasiados archivos en los directorios monitorizados).
`monitorExtensions` configura qué extensiones de archivo deben monitorizarse en los directorios `monitorDir`.
Si `options.enable_file_monitor` es `true`, se activa la monitorización de actualizaciones de archivos (en Linux se activa por defecto en modo debug).
Si `options.enable_memory_monitor` es `true`, se activa la monitorización de uso de memoria (no compatible con Windows).

> **Consejo**
> En Windows, la monitorización de archivos solo se activa al ejecutar `windows.bat` o `php windows.php`.
