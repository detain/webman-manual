# Processus personnalisés

Dans webman, vous pouvez personnaliser les listeners ou les processus comme dans workerman.

> **Remarque**
> Les utilisateurs Windows doivent démarrer webman avec `php windows.php` pour exécuter des processus personnalisés.

## Service HTTP personnalisé
Il arrive que vous ayez besoin de modifier le code principal du service HTTP de webman. Dans ce cas, vous pouvez utiliser un processus personnalisé.

Créez par exemple le fichier `app\Server.php`.

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // Réécrivez ici les méthodes de Webman\App
}
```

Ajoutez la configuration suivante dans `config/process.php`.

```php
use Workerman\Worker;

return [
    // ... autres configurations omises ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Nombre de processus
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Classe de requête
            'logger' => \support\Log::channel('default'), // Instance de journalisation
            'appPath' => app_path(), // Emplacement du répertoire app
            'publicPath' => public_path() // Emplacement du répertoire public
        ]
    ]
];
```

> **Conseil**
> Pour désactiver le processus HTTP intégré de webman, définissez simplement `listen=>''` dans `config/server.php`.

## Exemple de listener WebSocket personnalisé

Créez `app/Pusher.php`.

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

> Remarque : Toutes les méthodes onXXX doivent être publiques.

Ajoutez la configuration suivante dans `config/process.php`.

```php
return [
    // ... autres configurations de processus omises ...
    
    // websocket_test est le nom du processus
    'websocket_test' => [
        // Classe du processus, c'est-à-dire la classe Pusher définie ci-dessus
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## Exemple de processus sans écoute

Créez `app/TaskTest.php`.

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // Vérifier la base de données toutes les 10 secondes pour les nouvelles inscriptions
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

Ajoutez la configuration suivante dans `config/process.php`.

```php
return [
    // ... autres configurations de processus omises ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> Remarque : Si listen est omis, le processus n’écoute sur aucun port. Si count est omis, le nombre de processus par défaut est 1.

## Explication du fichier de configuration

Une configuration complète de processus est définie comme suit :

```php
return [
    // ... 

    // websocket_test est le nom du processus
    'websocket_test' => [
        // Classe du processus
        'handler' => app\Pusher::class,
        // Protocole, IP et port d’écoute (optionnel)
        'listen'  => 'websocket://0.0.0.0:8888',
        // Nombre de processus (optionnel, par défaut 1)
        'count'   => 2,
        // Utilisateur d’exécution (optionnel, par défaut l’utilisateur actuel)
        'user'    => '',
        // Groupe d’exécution (optionnel, par défaut le groupe actuel)
        'group'   => '',
        // Prise en charge du rechargement (optionnel, par défaut true)
        'reloadable' => true,
        // Activer reusePort
        'reusePort'  => true,
        // Transport (optionnel, mettre 'ssl' si SSL requis, par défaut 'tcp')
        'transport'  => 'tcp',
        // Contexte (optionnel, chemin du certificat si transport est 'ssl')
        'context'    => [], 
        // Paramètres du constructeur de la classe de processus (optionnel)
        'constructor' => [],
        // Indique si ce processus est activé
        'enable' => true
    ],
];
```

## Conclusion

Les processus personnalisés de webman sont une encapsulation simple de workerman. Ils séparent la configuration de la logique métier et implémentent les callbacks `onXXX` de workerman via des méthodes de classe. Le reste de l’utilisation est identique à workerman.
