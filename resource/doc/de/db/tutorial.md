# Datenbank Schnellstart (basierend auf Laravel-Datenbankkomponente)

[webman/database](https://github.com/webman-php/database) baut auf [illuminate/database](https://github.com/illuminate/database) auf und ergänzt Connection-Pooling für Koroutinen- und Nicht-Koroutinen-Umgebungen. Die Verwendung entspricht Laravel.

Zusätzlich können Sie das Kapitel [Verwendung anderer Datenbankkomponenten](others.md) nutzen, um ThinkPHP oder andere Datenbanken zu verwenden.

## Datenbank-Installation

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

Nach der Installation ist ein Neustart erforderlich (reload hat keine Wirkung).

> **Hinweis**
> webman/database hängt von Laravels `illuminate/database` ab, daher werden die Abhängigkeiten von `illuminate/database` automatisch mitinstalliert.

> **Achtung**
> Wenn Sie keine Paginierung, keine DB-Events und kein SQL-Logging brauchen, reicht:
> `composer require -W webman/database`

## Datenbank-Konfiguration
`config/database.php`
```php

return [
    // Standard-Datenbank
    'default' => 'mysql',

    // Verbindungs-Konfigurationen
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'test',
            'username'    => 'root',
            'password'    => '',
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
            'options' => [
                PDO::ATTR_EMULATE_PREPARES => false, // Erforderlich bei swoole oder swow als Runtime
            ],
            'pool' => [ // Connection-Pool-Konfiguration
                'max_connections' => 5, // Maximale Verbindungszahl
                'min_connections' => 1, // Minimale Verbindungszahl
                'wait_timeout' => 3,    // Max. Wartezeit auf Verbindung aus dem Pool; danach Exception. Nur bei Koroutinen
                'idle_timeout' => 60,   // Max. Leerlaufzeit von Verbindungen; danach Schließen bis min_connections
                'heartbeat_interval' => 50, // Pool-Heartbeat-Intervall (Sekunden); unter 60 Sekunden empfohlen
            ],
        ],
    ],
];
```

Bis auf die `pool`-Konfiguration ist alles wie bei Laravel.

## Zum Connection Pool
* Jeder Prozess hat seinen eigenen Connection Pool, Pools werden nicht zwischen Prozessen geteilt.
* Ohne Koroutinen werden Anfragen nacheinander ausgeführt, es gibt keine Parallelität, daher maximal eine Verbindung im Pool.
* Mit Koroutinen laufen Anfragen parallel; der Pool passt die Verbindungszahl dynamisch an, maximal `max_connections`, mindestens `min_connections`.
* Da der Pool maximal `max_connections` hat, warten bei mehr DB-Koroutinen Überschüsse bis zu `wait_timeout` Sekunden; danach wird eine Exception ausgelöst.
* Im Leerlauf (mit und ohne Koroutinen) werden Verbindungen nach `idle_timeout` zurückgegeben, bis `min_connections` erreicht ist (`min_connections` kann 0 sein).


## Datenbank-Beispiel
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        $default_uid = 29;
        $uid = $request->get('uid', $default_uid);
        $name = Db::table('users')->where('uid', $uid)->value('username');
        return response("hello $name");
    }
}
```

Die Nutzung entspricht Laravel: Mit der Methode `Db::table()` arbeiten Sie mit der Datenbank.
