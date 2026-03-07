# Processi personalizzati

In webman puoi personalizzare listener o processi come in workerman.

> **Nota**
> Gli utenti Windows devono avviare webman con `php windows.php` per eseguire processi personalizzati.

## Server HTTP personalizzato
A volte potresti aver bisogno di modificare il codice principale del servizio HTTP di webman. In tal caso puoi usare un processo personalizzato.

Ad esempio, crea il file `app\Server.php`.

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // Qui sovrascrivi i metodi di Webman\App
}
```

Aggiungi la seguente configurazione in `config/process.php`.

```php
use Workerman\Worker;

return [
    // ... altre configurazioni omesse ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Numero di processi
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Classe della richiesta
            'logger' => \support\Log::channel('default'), // Istanza del logger
            'appPath' => app_path(), // Posizione della directory app
            'publicPath' => public_path() // Posizione della directory public
        ]
    ]
];
```

> **Suggerimento**
> Per disattivare il processo HTTP predefinito di webman, imposta semplicemente `listen=>''` in `config/server.php`.

## Esempio di listener WebSocket personalizzato

Crea `app/Pusher.php`.

```php
<?php
namespace app;

use Workerman\Connection\TcpConnection;

class Pusher
{
    public function onConnect(TcpConnection $connection)
    {
        echo "onConnect\n";
    }

    public function onWebSocketConnect(TcpConnection $connection, $http_buffer)
    {
        echo "onWebSocketConnect\n";
    }

    public function onMessage(TcpConnection $connection, $data)
    {
        $connection->send($data);
    }

    public function onClose(TcpConnection $connection)
    {
        echo "onClose\n";
    }
}
```

> Nota: tutti i metodi onXXX devono essere pubblici.

Aggiungi la seguente configurazione in `config/process.php`.

```php
return [
    // ... altre configurazioni di processo omesse ...
    
    // websocket_test è il nome del processo
    'websocket_test' => [
        // Qui si specifica la classe del processo, cioè la classe Pusher definita sopra
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## Esempio di processo senza ascolto

Crea `app/TaskTest.php`.

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // Controllare ogni 10 secondi se ci sono nuovi utenti registrati
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

Aggiungi la seguente configurazione in `config/process.php`.

```php
return [
    // ... altre configurazioni di processo omesse ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> Nota: se listen viene omesso, il processo non ascolta su nessuna porta; se count viene omesso, il numero di processi predefinito è 1.

## Spiegazione del file di configurazione

Una configurazione completa di processo è definita come segue:

```php
return [
    // ... 

    // websocket_test è il nome del processo
    'websocket_test' => [
        // Qui si specifica la classe del processo
        'handler' => app\Pusher::class,
        // Protocollo, IP e porta da ascoltare (opzionale)
        'listen'  => 'websocket://0.0.0.0:8888',
        // Numero di processi (opzionale, predefinito 1)
        'count'   => 2,
        // Utente di esecuzione del processo (opzionale, predefinito utente corrente)
        'user'    => '',
        // Gruppo di esecuzione del processo (opzionale, predefinito gruppo corrente)
        'group'   => '',
        // Se il processo supporta il ricaricamento (opzionale, predefinito true)
        'reloadable' => true,
        // Abilitare reusePort
        'reusePort'  => true,
        // Trasporto (opzionale, impostare 'ssl' se richiesto SSL, predefinito 'tcp')
        'transport'  => 'tcp',
        // Contesto (opzionale, passare percorso certificato se transport è 'ssl')
        'context'    => [], 
        // Parametri del costruttore della classe del processo (opzionale)
        'constructor' => [],
        // Se questo processo è abilitato
        'enable' => true
    ],
];
```

## Conclusione

I processi personalizzati di webman sono essenzialmente una semplice incapsulazione di workerman. Separa la configurazione dalla logica di business e implementa i callback `onXXX` di workerman tramite metodi di classe. Il resto dell’uso è identico a workerman.
