# Guida rapida al database (componente Laravel)

[webman/database](https://github.com/webman-php/database) si basa su [illuminate/database](https://github.com/illuminate/database) e aggiunge il connection pooling per ambienti con e senza coroutine. L’uso è identico a Laravel.

Puoi anche consultare [Uso di altri componenti database](others.md) per usare ThinkPHP o altri database.

## Installazione del database

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

Dopo l’installazione è necessario eseguire il restart (reload non è sufficiente).

> **Suggerimento**
> webman/database dipende da `illuminate/database` di Laravel, quindi le dipendenze vengono installate automaticamente.

> **Nota**
> Se non ti servono paginazione, eventi database o logging SQL, basta eseguire:
> `composer require -W webman/database`

## Configurazione del database
`config/database.php`
```php

return [
    // Database predefinito
    'default' => 'mysql',

    // Configurazioni delle connessioni
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
                PDO::ATTR_EMULATE_PREPARES => false, // Obbligatorio con swoole o swow
            ],
            'pool' => [ // Configurazione del connection pool
                'max_connections' => 5, // Numero massimo di connessioni
                'min_connections' => 1, // Numero minimo di connessioni
                'wait_timeout' => 3,    // Tempo massimo di attesa per una connessione dal pool; superato → eccezione. Solo con coroutine
                'idle_timeout' => 60,   // Tempo massimo di inattività; poi chiusura fino a min_connections
                'heartbeat_interval' => 50, // Intervallo heartbeat del pool in secondi; consigliato < 60
            ],
        ],
    ],
];
```

Eccetto `pool`, il resto è uguale a Laravel.

## Sul connection pool
* Ogni processo ha il proprio pool; i pool non sono condivisi tra processi.
* Senza coroutine le richieste vengono eseguite in sequenza, nessuna concorrenza, quindi al massimo una connessione nel pool.
* Con coroutine le richieste sono parallele; il pool regola dinamicamente il numero di connessioni, senza superare `max_connections` né scendere sotto `min_connections`.
* Dato che il pool ha al massimo `max_connections`, quando le coroutine che usano il database sono di più, alcune attendono in coda fino a `wait_timeout` secondi; oltre scatta un’eccezione.
* In idle (con o senza coroutine), le connessioni vengono recuperate dopo `idle_timeout` fino a raggiungere `min_connections` (`min_connections` può essere 0).


## Esempio d’uso del database
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

L’uso è uguale a Laravel: il metodo `Db::table()` per operare sul database.
