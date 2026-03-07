# Componente ENV vlucas/phpdotenv

## Descrizione
`vlucas/phpdotenv` è un componente per il caricamento delle variabili d'ambiente, usato per distinguere la configurazione tra ambienti diversi (sviluppo, test, ecc.).

## Repository del progetto

https://github.com/vlucas/phpdotenv
  
## Installazione
 
```php
composer require vlucas/phpdotenv
 ```
  
## Utilizzo

### Creare un file `.env` nella directory root del progetto
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### Modificare il file di configurazione
**config/database.php**
```php
return [
    // Database predefinito
    'default' => 'mysql',

    // Configurazioni dei vari database
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

> **Suggerimento**
> Si consiglia di aggiungere il file `.env` alla lista `.gitignore` per evitare di committarlo. Aggiungete un file di esempio `.env.example` nel repository. In fase di deploy, copiate `.env.example` come `.env` e adattate la configurazione all'ambiente corrente. Così il progetto caricherà configurazioni diverse per ambiente.

> **Nota**
> `vlucas/phpdotenv` può avere bug con PHP in versione TS (Thread Safe). Usate la versione NTS (Non-Thread-Safe). La versione attuale di PHP si può verificare eseguendo `php -v`.

## Ulteriori informazioni

Visitate https://github.com/vlucas/phpdotenv
  
