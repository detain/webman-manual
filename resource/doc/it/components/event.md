# Gestione degli eventi
`webman/event` fornisce un meccanismo di eventi raffinato che consente di eseguire logica di business senza modificare il codice, realizzando il disaccoppiamento tra i moduli. Scenario tipico: quando un nuovo utente si registra con successo, basta pubblicare un evento personalizzato come `user.register`, e ogni modulo può ricevere l'evento ed eseguire la logica corrispondente.

## Installazione
`composer require webman/event`

## Sottoscrizione degli eventi
La sottoscrizione agli eventi si configura in modo centralizzato nel file `config/event.php`.

```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...altre funzioni di gestione eventi...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...altre funzioni di gestione eventi...
    ]
];
```

**Nota:**
- `user.register`, `user.logout`, ecc. sono nomi di eventi (stringa). Si consiglia l'uso di parole in minuscolo separate da punto (`.`).
- Un evento può avere più funzioni di gestione, chiamate nell'ordine di configurazione.

## Funzioni di gestione eventi
Le funzioni di gestione possono essere metodi di classe, funzioni o closure.

Esempio: creare la classe `app/event/User.php` (creare la cartella se non esiste).

```php
<?php
namespace app\event;
class User
{
    function register($user)
    {
        var_export($user);
    }
 
    function logout($user)
    {
        var_export($user);
    }
}
```

## Pubblicazione eventi
Usare `Event::dispatch($event_name, $data);` o `Event::emit($event_name, $data);` per pubblicare un evento. Ad esempio:

```php
<?php
namespace app\controller;
use support\Request;
use Webman\Event\Event;
class User
{
    public function register(Request $request)
    {
        $user = [
            'name' => 'webman',
            'age' => 2
        ];
        Event::dispatch('user.register', $user);
    }
}
```

Esistono due metodi per pubblicare: `Event::dispatch($event_name, $data);` e `Event::emit($event_name, $data);`, con gli stessi parametri. La differenza: `emit` intercetta le eccezioni internamente; se una funzione lancia un'eccezione, le altre continuano. Con `dispatch` le eccezioni non vengono intercettate; se una funzione lancia un'eccezione, l'esecuzione si interrompe e l'eccezione viene propagata.

> **Suggerimento**
> Il parametro `$data` può essere qualsiasi tipo di dato (array, istanza di classe, stringa, ecc.).

## Ascolto eventi con wildcard
La registrazione con wildcard permette di gestire più eventi con lo stesso listener. Ad esempio in `config/event.php`:

```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```

È possibile ottenere il nome concreto dell'evento tramite il secondo parametro `$event_data` della funzione di gestione:

```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // nome concreto dell'evento, es. user.register, user.logout, ecc.
        var_export($user);
    }
}
```

## Arrestare la diffusione dell'evento
Quando una funzione di gestione restituisce `false`, la diffusione dell'evento si interrompe.

## Gestione eventi con closure
La funzione di gestione può essere un metodo di classe o una closure. Ad esempio:

```php
<?php
return [
    'user.login' => [
        function($user){
            var_dump($user);
        }
    ]
];
```

## Visualizzare eventi e listener
Usare il comando `php webman event:list` per vedere tutti gli eventi e i listener configurati nel progetto.

## Ambito di supporto
Oltre al progetto principale, i [plugin base](../plugin/base.md) e i [plugin app](../app/app.md) supportano la configurazione `event.php`.
**File di configurazione plugin base:** `config/plugin/vendor/nome-plugin/event.php`
**File di configurazione plugin app:** `plugin/nome-plugin/config/event.php`

## Note
La gestione degli eventi non è asincrona e non è adatta a operazioni lente; queste andrebbero gestite con code di messaggi, ad esempio [webman/redis-queue](https://www.workerman.net/plugin/12).
