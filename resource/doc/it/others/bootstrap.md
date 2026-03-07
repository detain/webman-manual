# Inizializzazione dell'attività

A volte è necessario eseguire un'attività di inizializzazione dopo l'avvio di un processo. Questa inizializzazione viene eseguita solo una volta durante il ciclo di vita del processo, ad esempio impostare un timer dopo l'avvio del processo o inizializzare la connessione al database. Di seguito forniremo una spiegazione in merito.

## Principio
In base alle spiegazioni nel **[flusso di esecuzione](process.md)**, una volta avviato il processo, webman carica le classi definite in `config/bootstrap.php` (incluse le classi definite in `config/plugin/*/*/bootstrap.php`) ed esegue il metodo di avvio della classe. Possiamo inserire il codice dell'attività nell'implementazione del metodo di avvio, così da completare l'attività di inizializzazione dopo l'avvio del processo.

## Procedura
Supponiamo di voler creare un timer per l'invio periodico dell'utilizzo della memoria corrente del processo. Chiamiamo questa classe `MemReport`.

#### Esegui il comando
Esegui il comando `php webman make:bootstrap MemReport` per generare il file di inizializzazione `app/bootstrap/MemReport.php`

> **Suggerimento**
> Se `webman` non dispone del pacchetto `webman/console`, esegui il comando `composer require webman/console` per installarlo.

#### Modifica del file di inizializzazione
Modifica `app/bootstrap/MemReport.php` con un contenuto simile al seguente:
```php
<?php

namespace app\bootstrap;

use Webman\Bootstrap;

class MemReport implements Bootstrap
{
    public static function start($worker)
    {
        // E' un ambiente di linea di comando ?
        $is_console = !$worker;
        if ($is_console) {
            // Se non si desidera eseguire questa inizializzazione in un ambiente di linea di comando, basta restituire qui.
            return;
        }
        
        // Esegui ogni 10 secondi
        \Workerman\Timer::add(10, function () {
            // Per semplicità, qui usiamo la stampa anziché il processo di invio.
            echo memory_get_usage() . "\n";
        });
        
    }

}
```

> **Suggerimento**
> Quando si utilizza la riga di comando, il framework eseguirà il metodo di avvio definito in `config/bootstrap.php`. Possiamo determinare se ci troviamo in un ambiente di linea di comando basandoci sul valore di `$worker`, decidendo quindi se eseguire il codice di inizializzazione dell'attività.

#### Configurazione dell'avvio del processo
Apri `config/bootstrap.php` e aggiungi la classe `MemReport` all'elenco di avvio.
```php
return [
    // ... altre configurazioni sono omesse ...
    
    app\bootstrap\MemReport::class,
];
```

In questo modo, abbiamo completato un flusso di inizializzazione dell'attività.

## Ulteriori dettagli
Anche i [processi personalizzati](../process.md) eseguono il metodo di avvio definito in `config/bootstrap.php` dopo il loro avvio. Possiamo usare `$worker->name` per determinare quale processo è attualmente in esecuzione e inoltre `$worker->id` per identificare il numero del processo. In tal modo possiamo decidere se eseguire il codice di inizializzazione dell'attività in quel processo. Ad esempio, se dobbiamo eseguire solo nel processo 0 di webman, il contenuto di `MemReport.php` sarà simile al seguente:
```php
<?php

namespace app\bootstrap;

use Webman\Bootstrap;

class MemReport implements Bootstrap
{
    public static function start($worker)
    {
        // E' un ambiente di linea di comando ?
        $is_console = !$worker;
        if ($is_console) {
            // Se non si desidera eseguire questa inizializzazione in un ambiente di linea di comando, basta restituire qui.
            return;
        }
        
        // Esegui solo nel processo 0 di webman
        if ($worker->name != 'webman' || $worker->id != 0) {
            return;
        }
        
        // Esegui ogni 10 secondi
        \Workerman\Timer::add(10, function () {
            // Per semplicità, qui usiamo la stampa anziché il processo di invio
            echo memory_get_usage() . "\n";
        });
        
    }

}
```
