# Gestione delle operazioni lente

A volte è necessario gestire operazioni lente per evitare che influenzino il trattamento delle altre richieste in webman. Queste operazioni possono utilizzare diverse soluzioni di elaborazione a seconda della situazione.

## Opzione 1: Utilizzo di code di messaggi
Fare riferimento a [coda Redis](../queue/redis.md) [coda Stomp](../queue/stomp.md)

#### Vantaggi
Può gestire picchi improvvisi di richieste di elaborazione

#### Svantaggi
Non può restituire direttamente i risultati al client. Se è necessario inviare i risultati, è necessario coordinarsi con altri servizi, ad esempio utilizzando [webman/push](https://www.workerman.net/plugin/2) per inviare i risultati dell'elaborazione.

## Opzione 2: Aggiunta di una nuova porta HTTP

Aggiunta di una nuova porta HTTP per gestire le richieste lente. Queste richieste lente vengono elaborate accedendo a questa porta da un gruppo specifico di processi e i risultati vengono restituiti direttamente al client dopo l'elaborazione.

#### Vantaggi
Può restituire direttamente i dati al client

#### Svantaggi
Non è in grado di gestire picchi improvvisi di richieste

#### Procedura di implementazione
Aggiungere la seguente configurazione in `config/process.php`.
```php
return [
    // ... Altre configurazioni omesse qui ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Numero di processi
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Impostazione della classe di richiesta
            'logger' => \support\Log::channel('default'), // Istanza del logger
            'appPath' => app_path(), // Posizione della directory app
            'publicPath' => public_path() // Posizione della directory public
        ]
    ]
];
```

In questo modo, le interfacce lente possono passare attraverso il gruppo di processi su `http://127.0.0.1:8686/` senza influenzare l'elaborazione delle altre operazioni negli altri processi.

Per rendere impercettibile la differenza delle porte al frontend, è possibile aggiungere un proxy alla porta 8686 in nginx. Supponendo che i percorsi delle richieste lente inizino con `/task`, la configurazione nginx sarebbe simile alla seguente:
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# Aggiungere un nuovo upstream 8686
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # Le richieste che iniziano con /task vanno alla porta 8686, sostituire /task con il prefisso desiderato
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # Le altre richieste vanno alla porta 8787 originale
  location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      if (!-f $request_filename){
          proxy_pass http://webman;
      }
  }
}
```

In questo modo, quando il client accede a `dominio.com/task/xxx`, la richiesta verrà elaborata dalla porta 8686 separata senza influenzare l'elaborazione delle richieste sulla porta 8787.

## Opzione 3: Utilizzo di HTTP Chunked per invio asincrono di dati segmentati

#### Vantaggi
Può restituire direttamente i dati al client

**Installare workerman/http-client**

```
composer require workerman/http-client
```

**app/controller/IndexController.php**
```php
<?php
namespace app\controller;

use support\Request;
use support\Response;
use Workerman\Protocols\Http\Chunk;

class IndexController
{
    public function index(Request $request)
    {
        $connection = $request->connection;
        $http = new \Workerman\Http\Client();
        $http->get('https://example.com/', function ($response) use ($connection) {
            $connection->send(new Chunk($response->getBody()));
            $connection->send(new Chunk('')); // Inviare chunk vuoto per indicare la fine della risposta
        });
        // Inviare prima gli header HTTP, i dati vengono inviati in modo asincrono
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **Nota**
> Questo esempio utilizza il client `workerman/http-client` per recuperare i risultati HTTP in modo asincrono e restituire i dati. È possibile utilizzare anche altri client asincroni come [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html).
