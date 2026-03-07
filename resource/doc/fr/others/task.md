# Traitement des opérations lentes

Parfois, nous devons traiter des opérations lentes pour éviter qu'elles n'affectent le traitement des autres requêtes dans webman. Ces opérations peuvent être traitées selon différentes solutions selon la situation.

## Option 1 : Utilisation de files d'attente de messages
Voir [file d'attente Redis](../queue/redis.md) [file d'attente Stomp](../queue/stomp.md)

#### Avantages
Peut gérer des pics soudains de demandes de traitement

#### Inconvénients
Impossible de retourner directement les résultats au client. Si la diffusion des résultats est nécessaire, elle doit être coordonnée avec d'autres services, par exemple l'utilisation de [webman/push](https://www.workerman.net/plugin/2) pour diffuser les résultats du traitement.

## Option 2 : Ajout d'un nouveau port HTTP

Ajout d'un nouveau port HTTP pour traiter les requêtes lentes. Ces requêtes lentes sont traitées par un groupe spécifique de processus via l'accès à ce port, et les résultats sont renvoyés directement au client après le traitement.

#### Avantages
Peut renvoyer directement les données au client

#### Inconvénients
Impossible de gérer des pics soudains de demandes

#### Étapes de mise en œuvre
Ajoutez la configuration suivante dans `config/process.php`.
```php
return [
    // ... Autres configurations omises ici ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Nombre de processus
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Configuration de la classe de requête
            'logger' => \support\Log::channel('default'), // Instance du logger
            'appPath' => app_path(), // Emplacement du répertoire app
            'publicPath' => public_path() // Emplacement du répertoire public
        ]
    ]
];
```

Ainsi, les interfaces lentes peuvent transiter par le groupe de processus sur `http://127.0.0.1:8686/` sans affecter le traitement des autres processus.

Pour que le frontend ne perçoive pas la différence de port, vous pouvez ajouter un proxy vers le port 8686 dans nginx. Si les chemins des requêtes lentes commencent tous par `/task`, la configuration nginx serait similaire à ce qui suit :
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# Ajouter un nouvel upstream 8686
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # Les requêtes commençant par /task sont dirigées vers le port 8686, modifiez /task selon le préfixe nécessaire
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # Les autres requêtes passent par le port 8787 d'origine
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

Ainsi, lorsque le client accède à `domaine.com/task/xxx`, la requête sera traitée par le port 8686 distinct sans affecter le traitement des requêtes sur le port 8787.

## Option 3 : Utilisation de HTTP Chunked pour l'envoi asynchrone de données segmentées

#### Avantages
Peut renvoyer directement les données au client

**Installation de workerman/http-client**

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
            $connection->send(new Chunk('')); // Envoyer un chunk vide pour indiquer la fin de la réponse
        });
        // Envoyer d'abord les en-têtes HTTP, les données sont envoyées de manière asynchrone
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **Remarque**
> Cet exemple utilise le client `workerman/http-client` pour récupérer de manière asynchrone les résultats HTTP et renvoyer les données. Vous pouvez également utiliser d'autres clients asynchrones tels que [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html).
