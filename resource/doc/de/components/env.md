# ENV-Komponente vlucas/phpdotenv

## Beschreibung
`vlucas/phpdotenv` ist eine Komponente zum Laden von Umgebungsvariablen und dient dazu, Konfigurationen für verschiedene Umgebungen (wie Entwicklungsumgebung, Testumgebung usw.) zu unterscheiden.

## Projektadresse

https://github.com/vlucas/phpdotenv
  
## Installation
 
```php
composer require vlucas/phpdotenv
 ```
  
## Verwendung

### Erstellen Sie eine neue `.env`-Datei im Projektstammverzeichnis
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### Konfigurationsdatei anpassen
**config/database.php**
```php
return [
    // Standarddatenbank
    'default' => 'mysql',

    // Verschiedene Datenbankkonfigurationen
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **Hinweis**
> Es wird empfohlen, die `.env`-Datei in die `.gitignore`-Liste aufzunehmen, um sie nicht ins Code-Repository zu übernehmen. Fügen Sie stattdessen eine `.env.example`-Beispieldatei ins Repository ein. Beim Deployen des Projekts kopieren Sie `.env.example` als `.env` und passen die Konfiguration in `.env` an die aktuelle Umgebung an. So kann das Projekt je nach Umgebung unterschiedliche Konfigurationen laden.

> **Achtung**
> `vlucas/phpdotenv` kann in der PHP-TS-Version (Thread-Safe) Fehler verursachen. Verwenden Sie die NTS-Version (Non-Thread-Safe). Die aktuelle PHP-Version können Sie mit `php -v` prüfen.

## Weitere Informationen

Besuchen Sie https://github.com/vlucas/phpdotenv
  
