# Medoo-Datenbank

[webman/medoo](https://github.com/webman-php/medoo) erweitert [Medoo](https://medoo.in/) um Verbindungs-Pool-Unterstützung und funktioniert sowohl in Koroutinen- als auch in Nicht-Koroutinen-Umgebungen. Die Verwendung entspricht Medoo.

## Installation
`composer require webman/medoo`

## Medoo-Datenbankkonfiguration
Konfigurationsdatei: `config/plugin/webman/medoo/database.php`

## Medoo-Datenbankverwendung
```php
<?php
namespace app\controller;

use support\Request;
use support\Medoo;

class Index
{
    public function index(Request $request)
    {
        $user = Medoo::get('user', '*', ['uid' => 1]);
        return json($user);
    }
}
```

> **Hinweis**
> `Medoo::get('user', '*', ['uid' => 1]);`
> entspricht
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## Medoo-Mehfach-Datenbankkonfiguration

**Konfiguration**
Fügen Sie in `config/plugin/webman/medoo/database.php` eine neue Konfiguration hinzu; der Schlüssel ist beliebig, hier wird `other` verwendet.

```php
<?php
return [
    'default' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [ // Verbindungspool-Konfiguration
            'max_connections' => 5, // Maximale Verbindungsanzahl
            'min_connections' => 1, // Minimale Verbindungsanzahl
            'wait_timeout' => 60,   // Maximale Wartezeit beim Entnehmen aus dem Pool; bei Überschreitung wird eine Ausnahme ausgelöst
            'idle_timeout' => 3,    // Maximale Leerlaufzeit von Verbindungen im Pool; überschrittene werden geschlossen, bis min_connections erreicht ist
            'heartbeat_interval' => 50, // Herzschlagintervall des Pools in Sekunden; empfohlen unter 60 Sekunden
        ]
    ],
    // Hier wird eine neue Konfiguration 'other' hinzugefügt
    'other' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [
            'max_connections' => 5,
            'min_connections' => 1,
            'wait_timeout' => 60,
            'idle_timeout' => 3,
            'heartbeat_interval' => 50,
        ],
    ],
];
```

## Medoo-Datenbankverwendung
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

Siehe [offizielle Medoo-Dokumentation](https://medoo.in/api/select)
