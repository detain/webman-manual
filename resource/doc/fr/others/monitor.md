# Surveillance des processus
webman intègre un processus de surveillance qui prend en charge deux fonctions :
1. Surveille les mises à jour de fichiers et recharge automatiquement le nouveau code métier (généralement en développement).
2. Surveille l'utilisation mémoire de tous les processus ; si un processus est sur le point de dépasser la limite `memory_limit` définie dans `php.ini`, il est redémarré automatiquement en toute sécurité (sans impact sur l'activité).

## Configuration de la surveillance
Configuration de `monitor` dans `config/process.php` :
```php

global $argv;

return [
    // Détection des mises à jour de fichiers et rechargement automatique
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // Surveiller ces répertoires
            'monitorDir' => array_merge([    // Quels répertoires doivent être surveillés
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Fichiers avec ces extensions seront surveillés
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // Activer la surveillance des fichiers
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // Activer la surveillance mémoire
            ]
        ]
    ]
];
```
`monitorDir` configure les répertoires dont les mises à jour sont surveillées (ne pas avoir trop de fichiers dans les répertoires surveillés).
`monitorExtensions` configure les extensions de fichiers à surveiller dans les répertoires `monitorDir`.
Si `options.enable_file_monitor` est `true`, la surveillance des mises à jour de fichiers est activée (par défaut en mode debug sous Linux).
Si `options.enable_memory_monitor` est `true`, la surveillance mémoire est activée (non prise en charge sous Windows).

> **Conseil**
> Sous Windows, la surveillance des fichiers n'est activée qu'en exécutant `windows.bat` ou `php windows.php`.
