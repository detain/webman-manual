# Coda Redis

Coda di messaggi basata su Redis che supporta l'elaborazione ritardata dei messaggi.

## Installazione
`composer require webman/redis-queue`

## File di configurazione
Il file di configurazione Redis viene generato automaticamente in `{progetto-principale}/config/plugin/webman/redis-queue/redis.php`, con contenuto simile al seguente:
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // Password, opzionale
            'db' => 0,            // Database
            'max_attempts'  => 5, // Numero di tentativi dopo fallimento di consumo
            'retry_seconds' => 5, // Intervallo di ritentativo in secondi
        ]
    ],
];
```

### Ritentativi in caso di fallimento di consumo
Se il consumo fallisce (si verifica un'eccezione), il messaggio viene inserito nella coda ritardata e attende il ritentativo successivo. Il numero di ritentativi è controllato da `max_attempts`, l'intervallo da `retry_seconds` e `max_attempts` congiuntamente. Es. se `max_attempts` è 5 e `retry_seconds` è 10, l'intervallo del 1° ritentativo è `1*10` secondi, del 2° `2*10` secondi, del 3° `3*10` secondi, fino a 5 ritentativi. Se si supera il numero di ritentativi impostato in `max_attempts`, il messaggio va nella coda falliti con chiave `{redis-queue}-failed`.

## Invio messaggi (sincrono)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // Nome della coda
        $queue = 'send-mail';
        // Dati, si può passare un array direttamente senza serializzazione
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Inviare messaggio
        Redis::send($queue, $data);
        // Inviare messaggio ritardato, elaborato dopo 60 secondi
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
In caso di invio riuscito, `Redis::send()` restituisce true, altrimenti false o lancia un'eccezione.

> **Suggerimento**
> Può esserci scostamento nel tempo di consumo della coda ritardata. Es. quando la velocità di consumo è inferiore a quella di produzione, la coda può accumularsi e ritardare il consumo. Mitigazione: avviare più processi consumatori.

## Invio messaggi (asincrono)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // Nome della coda
        $queue = 'send-mail';
        // Dati, si può passare un array direttamente senza serializzazione
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Inviare messaggio
        Client::send($queue, $data);
        // Inviare messaggio ritardato, elaborato dopo 60 secondi
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` non restituisce alcun valore. È un push asincrono e non garantisce il 100% di consegna a Redis.

> **Suggerimento**
> Il principio di `Client::send()` è creare una coda in memoria locale e sincronizzare i messaggi in modo asincrono con Redis (la sincronizzazione è veloce, circa 10.000 messaggi al secondo). Se il processo si riavvia prima che i dati della coda in memoria siano sincronizzati del tutto, possono andare persi messaggi. L'invio asincrono con `Client::send()` è adatto per messaggi non critici.

> **Suggerimento**
> `Client::send()` è asincrono e può essere usato solo nell'ambiente di esecuzione Workerman. Per script da riga di comando usare l'interfaccia sincrona `Redis::send()`.

## Inviare messaggi da altri progetti
A volte è necessario inviare messaggi da altri progetti e non si può usare `webman\redis-queue`. In tali casi si può riferire alla seguente funzione per inviare messaggi alla coda.

```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting';
    $queue_delay = '{redis-queue}-delayed';
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```

Qui il parametro `$redis` è l'istanza Redis. Es. l'uso dell'estensione redis è simile a:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## Consumo
Il file di configurazione del processo consumatore è in `{progetto-principale}/config/plugin/webman/redis-queue/process.php`. La directory dei consumatori è sotto `{progetto-principale}/app/queue/redis/`.

Eseguendo il comando `php webman redis-queue:consumer my-send-mail` si genera il file `{progetto-principale}/app/queue/redis/MyMailSend.php`.

> **Suggerimento**
> Questo comando richiede l'installazione del plugin [Console](../plugin/console.md). Se non si vuole installarlo, si può creare manualmente un file simile al seguente:

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // Nome della coda da consumare
    public $queue = 'send-mail';

    // Nome connessione, corrisponde alla connessione in plugin/webman/redis-queue/redis.php
    public $connection = 'default';

    // Consumo
    public function consume($data)
    {
        // Non serve deserializzare
        var_export($data); // Output ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // Callback in caso di fallimento consumo
    /* 
    $package = [
        'id' => 1357277951, // ID messaggio
        'time' => 1709170510, // Tempo messaggio
        'delay' => 0, // Tempo ritardo
        'attempts' => 2, // Conteggio consumi
        'queue' => 'send-mail', // Nome coda
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // Contenuto messaggio
        'max_attempts' => 5, // Numero max ritentativi
        'error' => 'Messaggio errore' // Messaggio errore
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // Non serve deserializzare
        var_export($package); 
    }
}
```

> **Nota**
> Il consumo è considerato riuscito se durante il consumo non viene lanciata eccezione o Error; altrimenti fallimento e il messaggio entra nella coda di ritentativi. redis-queue non ha meccanismo ack; si può considerare come ack automatico (quando non c'è eccezione o Error). Per marcare il messaggio corrente come non consumato con successo, si può lanciare manualmente un'eccezione per mandare il messaggio nella coda di ritentativi. In pratica non è diverso da un meccanismo ack.

> **Suggerimento**
> I consumatori supportano più server e processi, e lo stesso messaggio **non** viene consumato due volte. I messaggi consumati vengono rimossi automaticamente dalla coda; non serve eliminazione manuale.

> **Suggerimento**
> I processi consumatori possono consumare più code diverse contemporaneamente. Aggiungere una nuova coda non richiede modificare la configurazione in `process.php`. Per aggiungere un consumatore di coda nuova, basta aggiungere la classe `Consumer` corrispondente sotto `app/queue/redis` e usare la proprietà `$queue` per specificare il nome della coda da consumare.

> **Suggerimento**
> Gli utenti Windows devono eseguire `php windows.php` per avviare webman, altrimenti il processo consumatore non si avvierà.

> **Suggerimento**
> Il callback onConsumeFailure viene attivato a ogni fallimento di consumo. Qui si può gestire la logica post-fallimento. (Questa funzione richiede `webman/redis-queue>=1.3.2` e `workerman/redis-queue>=1.2.1`)

## Impostare processi consumatori diversi per code diverse
Di default tutti i consumatori condividono lo stesso processo. A volte si vuole separare il consumo di alcune code—es. business a consumo lento in un gruppo di processi, a consumo rapido in un altro. Per questo si possono dividere i consumatori in due directory, es. `app_path() . '/queue/redis/fast'` e `app_path() . '/queue/redis/slow'` (nota: il namespace della classe consumatore va aggiornato di conseguenza). La configurazione è:
```php
return [
    ...altre configurazioni omesse...
    
    'redis_consumer_fast'  => [ // La chiave è personalizzata, nessun vincolo di formato, qui redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Directory classi consumatori
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // La chiave è personalizzata, nessun vincolo di formato, qui redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Directory classi consumatori
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

In questo modo i consumatori veloci vanno nella directory `queue/redis/fast` e i lenti in `queue/redis/slow`, raggiungendo l'obiettivo di assegnare processi consumatori alle code.

## Configurazione Redis multipla
#### Configurazione
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // Password, tipo stringa, opzionale
            'db' => 0,           // Database
            'max_attempts'  => 5, // Ritentativi dopo fallimento consumo
            'retry_seconds' => 5, // Intervallo ritentativo in secondi
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // Password, tipo stringa, opzionale
            'db' => 0,           // Database
            'max_attempts'  => 5, // Ritentativi dopo fallimento consumo
            'retry_seconds' => 5, // Intervallo ritentativo in secondi
        ]
    ],
];
```

Nota: nella configurazione è stata aggiunta un'ulteriore configurazione Redis con chiave `other`.

#### Invio messaggi a Redis multipli

```php
// Inviare messaggio alla coda con chiave `default`
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// Uguale a
Client::send($queue, $data);
Redis::send($queue, $data);

// Inviare messaggio alla coda con chiave `other`
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### Consumo da Redis multipli
Consumare messaggi dalla coda con chiave `other` nella configurazione:
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // Nome della coda da consumare
    public $queue = 'send-mail';

    // === Impostare qui 'other' per consumare dalla coda con chiave 'other' nella configurazione ===
    public $connection = 'other';

    // Consumo
    public function consume($data)
    {
        // Non serve deserializzare
        var_export($data);
    }
}
```

## FAQ

**Perché compare l'errore `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)`?**

Questo errore si verifica solo con l'interfaccia di invio asincrono `Client::send()`. L'invio asincrono salva prima i messaggi in memoria locale, poi li invia a Redis quando il processo è inattivo. Se Redis riceve i messaggi più lentamente di quanto vengono prodotti, o il processo è occupato con altre attività e non ha tempo sufficiente per sincronizzare i messaggi dalla memoria a Redis, può verificarsi un accumulo. Se i messaggi restano accumulati per più di 600 secondi, viene attivato questo errore.

Soluzione: usare l'interfaccia di invio sincrono `Redis::send()` per l'invio messaggi.
