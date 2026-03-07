# Benutzerdefinierter Prozess

In webman können Sie wie in Workerman benutzerdefinierte Listener oder Prozesse erstellen.

> **Hinweis**
> Windows-Benutzer müssen `php windows.php` verwenden, um webman zu starten und benutzerdefinierte Prozesse auszuführen.

## Benutzerdefinierter HTTP-Server
Manchmal haben Sie möglicherweise spezielle Anforderungen, die eine Änderung des Kerncodes des webman HTTP-Servers erfordern. In diesem Fall können Sie benutzerdefinierte Prozesse verwenden.

Erstellen Sie z.B. die neue Datei `app\Server.php`.

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // Hier überschreiben Sie die Methoden von Webman\App
}
```

Fügen Sie die folgende Konfiguration zu `config/process.php` hinzu.

```php
use Workerman\Worker;

return [
    // ... andere Konfigurationen ausgelassen ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Anzahl der Prozesse
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Request-Klasse setzen
            'logger' => \support\Log::channel('default'), // Logger-Instanz
            'appPath' => app_path(), // Speicherort des app-Verzeichnisses
            'publicPath' => public_path() // Speicherort des public-Verzeichnisses
        ]
    ]
];
```

> **Tipp**
> Um den eingebauten HTTP-Prozess von webman zu deaktivieren, setzen Sie einfach `listen=>''` in `config/server.php`.

## Beispiel für benutzerdefinierten WebSocket-Listener

Erstellen Sie `app/Pusher.php`.

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

> Hinweis: Alle onXXX-Methoden müssen public sein.

Fügen Sie die folgende Konfiguration zu `config/process.php` hinzu.

```php
return [
    // ... andere Prozesskonfigurationen ausgelassen ...
    
    // websocket_test ist der Prozessname
    'websocket_test' => [
        // Hier wird die Prozessklasse angegeben, d.h. die oben definierte Pusher-Klasse
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## Beispiel für benutzerdefinierten Nicht-Listener-Prozess

Erstellen Sie `app/TaskTest.php`.

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // Überprüfen Sie alle 10 Sekunden, ob neue Benutzer registriert wurden
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

Fügen Sie die folgende Konfiguration zu `config/process.php` hinzu.

```php
return [
    // ... andere Prozesskonfigurationen ausgelassen ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> Hinweis: Wenn listen weggelassen wird, lauscht der Prozess auf keinem Port. Wenn count weggelassen wird, ist die Standardanzahl der Prozesse 1.

## Erklärung der Konfigurationsdatei

Eine vollständige Prozesskonfiguration wird wie folgt definiert:

```php
return [
    // ... 

    // websocket_test ist der Prozessname
    'websocket_test' => [
        // Hier wird die Prozessklasse angegeben
        'handler' => app\Pusher::class,
        // Protokoll, IP und Port zum Lauschen (optional)
        'listen'  => 'websocket://0.0.0.0:8888',
        // Anzahl der Prozesse (optional, Standard 1)
        'count'   => 2,
        // Benutzer, unter dem der Prozess läuft (optional, Standard aktueller Benutzer)
        'user'    => '',
        // Benutzergruppe, unter der der Prozess läuft (optional, Standard aktuelle Benutzergruppe)
        'group'   => '',
        // Unterstützt der aktuelle Prozess Reload? (optional, Standard true)
        'reloadable' => true,
        // reusePort aktivieren
        'reusePort'  => true,
        // Transport (optional, bei SSL-Bedarf auf ssl setzen, Standard tcp)
        'transport'  => 'tcp',
        // Kontext (optional, Zertifikatspfad übergeben, wenn transport ssl ist)
        'context'    => [], 
        // Konstruktorparameter der Prozessklasse (optional)
        'constructor' => [],
        // Ob dieser Prozess aktiviert ist
        'enable' => true
    ],
];
```

## Zusammenfassung

Benutzerdefinierte Prozesse in webman sind im Wesentlichen eine einfache Kapselung von Workerman. Sie trennen Konfiguration von Geschäftslogik und implementieren die `onXXX`-Callbacks von Workerman durch Klassenmethoden. Alle anderen Verwendungsarten sind mit Workerman identisch.
