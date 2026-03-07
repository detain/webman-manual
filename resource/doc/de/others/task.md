# Langsame Geschäftsverarbeitung

Manchmal müssen wir langsame Geschäftsprozesse bewältigen. Um zu vermeiden, dass langsame Geschäftsprozesse andere Anfrageverarbeitungen in webman beeinträchtigen, können je nach Situation unterschiedliche Verarbeitungslösungen eingesetzt werden.

## Lösung 1: Verwendung von Nachrichten-Warteschlangen
Siehe [Redis-Warteschlange](../queue/redis.md) [Stomp-Warteschlange](../queue/stomp.md)

#### Vorteile
Kann plötzliche Flut von Geschäftsverarbeitungsanfragen bewältigen

#### Nachteile
Ergebnisse können nicht direkt an den Client zurückgegeben werden. Wenn Ergebnisse übertragen werden müssen, ist die Zusammenarbeit mit anderen Diensten erforderlich, z. B. die Verwendung von [webman/push](https://www.workerman.net/plugin/2) zum Versenden der Verarbeitungsergebnisse.

## Lösung 2: Hinzufügen eines neuen HTTP-Ports

Hinzufügen eines neuen HTTP-Ports zur Verarbeitung langsamer Anfragen. Diese langsamen Anfragen werden durch Zugriff auf diesen Port von einer bestimmten Gruppe von Prozessen verarbeitet, und die Ergebnisse werden nach der Verarbeitung direkt an den Client zurückgegeben.

#### Vorteile
Daten können direkt an den Client zurückgegeben werden

#### Nachteile
Kann plötzliche Flut von Anfragen nicht bewältigen

#### Implementierungsschritte
Fügen Sie die folgende Konfiguration in `config/process.php` hinzu.
```php
return [
    // ... Andere Konfigurationen sind hier ausgelassen ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Anzahl der Prozesse
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Einstellung der Request-Klasse
            'logger' => \support\Log::channel('default'), // Logger-Instanz
            'appPath' => app_path(), // Speicherort des App-Verzeichnisses
            'publicPath' => public_path() // Speicherort des public-Verzeichnisses
        ]
    ]
];
```

So können langsame Schnittstellen über die Prozessgruppe unter `http://127.0.0.1:8686/` laufen, ohne die Geschäftsverarbeitung anderer Prozesse zu beeinträchtigen.

Damit das Frontend den Portunterschied nicht bemerkt, können Sie in nginx einen Proxy zum Port 8686 hinzufügen. Wenn die Pfade der langsamen Schnittstellenanfragen alle mit `/task` beginnen, sieht die nginx-Konfiguration ähnlich wie folgt aus:
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# Neuen 8686-Upstream hinzufügen
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # Anfragen, die mit /task beginnen, gehen an Port 8686; ändern Sie /task bei Bedarf in Ihr gewünschtes Präfix
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # Andere Anfragen gehen an den ursprünglichen Port 8787
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

So wird beim Client-Zugriff auf `domain.com/task/xxx` die Anfrage über den separaten Port 8686 verarbeitet, ohne die Anfrageverarbeitung auf Port 8787 zu beeinträchtigen.

## Lösung 3: Nutzung von HTTP Chunked für asynchrone segmentierte Datenubertragung

#### Vorteile
Daten können direkt an den Client zurückgegeben werden

**Installation von workerman/http-client**

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
            $connection->send(new Chunk('')); // Leeren Chunk senden, um Ende der Antwort zu signalisieren
        });
        // Zuerst HTTP-Header senden, Daten werden asynchron nachgeliefert
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **Hinweis**
> Dieses Beispiel verwendet den `workerman/http-client`, um HTTP-Ergebnisse asynchron abzurufen und zurückzugeben. Sie können auch andere asynchrone Clients verwenden, z. B. [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html).
