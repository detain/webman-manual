# Database Medoo

[webman/medoo](https://github.com/webman-php/medoo) estende [Medoo](https://medoo.in/) con il supporto del connection pool e funziona sia in ambiente coroutine che non-coroutine. L'utilizzo è identico a Medoo.

## Installazione
`composer require webman/medoo`

## Configurazione del database Medoo
Posizione del file di configurazione: `config/plugin/webman/medoo/database.php`

## Utilizzo del database Medoo
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

> **Suggerimento**
> `Medoo::get('user', '*', ['uid' => 1]);`
> equivale a
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## Configurazione di più database Medoo

**Configurazione**
Aggiungere una nuova configurazione in `config/plugin/webman/medoo/database.php` con una chiave qualsiasi; qui si utilizza `other`.

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
        'pool' => [ // Configurazione del connection pool
            'max_connections' => 5, // Numero massimo di connessioni
            'min_connections' => 1, // Numero minimo di connessioni
            'wait_timeout' => 60,   // Tempo massimo di attesa per ottenere una connessione dal pool; eccezione se superato
            'idle_timeout' => 3,    // Tempo massimo di inattività delle connessioni nel pool; quelle che lo superano vengono chiuse fino a min_connections
            'heartbeat_interval' => 50, // Intervallo di heartbeat del pool in secondi; si consiglia meno di 60 secondi
        ]
    ],
    // Aggiungere qui la configurazione 'other'
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

## Utilizzo del database Medoo
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

Consultare la [documentazione ufficiale di Medoo](https://medoo.in/api/select)
