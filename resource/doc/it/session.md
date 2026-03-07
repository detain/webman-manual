# Gestione delle sessioni webman

## Esempio
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

Ottieni un'istanza di `Workerman\Protocols\Http\Session` tramite `$request->session();` e usa i suoi metodi per aggiungere, modificare o eliminare i dati della sessione.

> **Nota**
> I dati della sessione vengono salvati automaticamente quando l'oggetto sessione viene distrutto.
> Se si memorizza l'oggetto sessione in una variabile globale, esso non verrà distrutto e non verrà salvato automaticamente. In tal caso è necessario chiamare manualmente `$session->save()` per salvare i dati.

## Ottenere tutti i dati della sessione
```php
$session = $request->session();
$all = $session->all();
```
Restituisce un array. Se non ci sono dati di sessione, restituisce un array vuoto.


## Ottenere un valore dalla sessione
```php
$session = $request->session();
$name = $session->get('name');
```
Se i dati non esistono, restituisce null.

È possibile passare un valore predefinito come secondo argomento al metodo get. Se il valore corrispondente non viene trovato nell'array di sessione, viene restituito il valore predefinito. Ad esempio:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## Salvare dati nella sessione
Usa il metodo `set` per salvare un singolo dato.
```php
$session = $request->session();
$session->set('name', 'tom');
```
Il metodo set non restituisce nulla. La sessione viene salvata automaticamente quando l'oggetto sessione viene distrutto.

Usa il metodo `put` per salvare più valori.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
Anche il metodo put non restituisce nulla.

## Eliminare dati dalla sessione
Usa il metodo `forget` per eliminare uno o più dati dalla sessione.
```php
$session = $request->session();
// Eliminare un elemento
$session->forget('name');
// Eliminare più elementi
$session->forget(['name', 'age']);
```

È disponibile anche il metodo `delete`. A differenza di `forget`, può eliminare solo un elemento.
```php
$session = $request->session();
// Equivalente a $session->forget('name');
$session->delete('name');
```


## Ottenere e eliminare un valore dalla sessione
```php
$session = $request->session();
$name = $session->pull('name');
```
È equivalente al seguente codice:
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
Se la sessione corrispondente non esiste, restituisce null.


## Eliminare tutti i dati della sessione
```php
$request->session()->flush();
```
Non restituisce nulla. I dati della sessione vengono eliminati automaticamente dallo storage quando l'oggetto sessione viene distrutto.


## Verificare se esiste un valore nella sessione
```php
$session = $request->session();
$has = $session->has('name');
```
Restituisce false se il valore della sessione non esiste o è null; altrimenti restituisce true.

```php
$session = $request->session();
$has = $session->exists('name');
```
Il codice sopra verifica anche l'esistenza di un valore nella sessione. La differenza è che `exists` restituisce true anche quando il valore è null.

## Funzione helper session()

webman fornisce la funzione helper `session()` per le stesse operazioni.
```php
// Ottenere l'istanza della sessione
$session = session();
// Equivalente a
$session = $request->session();

// Ottenere un valore
$value = session('key', 'default');
// Equivalente a
$value = session()->get('key', 'default');
// Equivalente a
$value = $request->session()->get('key', 'default');

// Assegnare valori alla sessione
session(['key1'=>'value1', 'key2' => 'value2']);
// Equivalente a
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// Equivalente a
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## File di configurazione
Il file di configurazione della sessione si trova in `config/session.php`. Il contenuto è simile a:
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class o RedisSessionHandler::class o RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // Se handler è FileSessionHandler::class, il valore è 'file',
    // se handler è RedisSessionHandler::class, il valore è 'redis',
    // se handler è RedisClusterSessionHandler::class, il valore è 'redis_cluster' (cluster Redis)
    'type'    => 'file',

    // Configurazioni diverse per handler diversi
    'config' => [
        // Configurazione per type 'file'
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // Configurazione per type 'redis'
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // Nome del cookie per memorizzare il session_id
    'auto_update_timestamp' => false,  // Aggiornare automaticamente la sessione, predefinito: disattivato
    'lifetime' => 7*24*60*60,          // Tempo di scadenza della sessione
    'cookie_lifetime' => 365*24*60*60, // Tempo di scadenza del cookie per il session_id
    'cookie_path' => '/',              // Percorso del cookie per il session_id
    'domain' => '',                    // Dominio del cookie per il session_id
    'http_only' => true,               // Abilitare httpOnly, predefinito: abilitato
    'secure' => false,                 // Abilitare la sessione solo su HTTPS, predefinito: disattivato
    'same_site' => '',                 // Prevenire attacchi CSRF e tracciamento utenti, valori: strict/lax/none
    'gc_probability' => [1, 1000],     // Probabilità di garbage collection della sessione
];
```

## Sicurezza
Non è consigliabile memorizzare direttamente istanze di classi nella sessione, in particolare di classi provenienti da fonti non attendibili. La deserializzazione può introdurre rischi per la sicurezza.

