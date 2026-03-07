# Мониторинг процессов
webman имеет встроенный процесс мониторинга, который поддерживает две функции:
1. Отслеживает обновления файлов и автоматически перезагружает новый бизнес-код (обычно используется при разработке).
2. Отслеживает использование памяти всеми процессами; если процесс близок к превышению лимита `memory_limit` из `php.ini`, он автоматически безопасно перезапускается (без влияния на работу).

## Настройка мониторинга
Конфигурация `monitor` в файле `config/process.php`:
```php

global $argv;

return [
    // Обнаружение обновлений файлов и автоматическая перезагрузка
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // Отслеживать эти каталоги
            'monitorDir' => array_merge([    // Какие каталоги нужно отслеживать
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Файлы с этими расширениями будут отслеживаться
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // Включить мониторинг файлов
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // Включить мониторинг памяти
            ]
        ]
    ]
];
```
`monitorDir` указывает, какие каталоги отслеживать на обновления (не стоит отслеживать слишком много файлов).
`monitorExtensions` указывает, какие расширения файлов в каталогах `monitorDir` отслеживать.
При `options.enable_file_monitor` = `true` включается мониторинг обновлений файлов (по умолчанию в режиме отладки под Linux).
При `options.enable_memory_monitor` = `true` включается мониторинг памяти (не поддерживается в Windows).

> **Совет**
> В Windows мониторинг файлов включается только при запуске `windows.bat` или `php windows.php`.
