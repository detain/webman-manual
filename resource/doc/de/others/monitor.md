# Prozessüberwachung
webman verfügt über einen eingebauten Monitor-Prozess, der zwei Funktionen unterstützt:
1. Überwacht Dateiänderungen und lädt den neuen Geschäftscode automatisch neu (üblicherweise während der Entwicklung).
2. Überwacht den Speicherverbrauch aller Prozesse; wenn ein Prozess kurz davor steht, das `memory_limit` aus der `php.ini` zu überschreiten, wird er automatisch sicher neu gestartet (ohne Auswirkungen auf den Betrieb).

## Überwachungskonfiguration
Konfiguration für `monitor` in `config/process.php`:
```php

global $argv;

return [
    // Erkennung von Dateiänderungen und automatisches Neuladen
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // Diese Verzeichnisse überwachen
            'monitorDir' => array_merge([    // Welche Verzeichnisse überwacht werden sollen
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Dateien mit diesen Endungen werden überwacht
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // Dateiüberwachung aktivieren
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // Speicherüberwachung aktivieren
            ]
        ]
    ]
];
```
`monitorDir` legt fest, welche Verzeichnisse auf Änderungen überwacht werden (die Anzahl der überwachten Dateien sollte nicht zu groß sein).
`monitorExtensions` legt fest, welche Dateiendungen in den `monitorDir`-Verzeichnissen überwacht werden.
Bei `options.enable_file_monitor` = `true` wird die Dateiüberwachung aktiviert (unter Linux standardmäßig im Debug-Modus aktiv).
Bei `options.enable_memory_monitor` = `true` wird die Speicherüberwachung aktiviert (unter Windows nicht unterstützt).

> **Hinweis**
> Unter Windows wird die Dateiüberwachung nur bei Ausführung von `windows.bat` oder `php windows.php` aktiviert.
