# Stomp-Warteschlange

Stomp ist ein einfaches (Streaming-)Text-Messaging-Protokoll, das ein interoperables Verbindungsformat bereitstellt und es STOMP-Clients ermöglicht, mit beliebigen STOMP-Nachrichtenbrokern zu kommunizieren. [workerman/stomp](https://github.com/walkor/stomp) implementiert den Stomp-Client und wird hauptsächlich für Nachrichtenwarteschlangen-Szenarien wie RabbitMQ, Apollo, ActiveMQ usw. verwendet.

## Installation
`composer require webman/stomp`

## Konfiguration
Die Konfigurationsdatei befindet sich unter `config/plugin/webman/stomp`

## Nachrichten senden
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // Warteschlange
        $queue = 'examples';
        // Daten (bei Array-Übergabe ist eine eigene Serialisierung erforderlich, z. B. mit json_encode, serialize usw.)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // Versand durchführen
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> Für Kompatibilität mit anderen Projekten bietet die Stomp-Komponente keine automatische Serialisierung und Deserialisierung. Bei Array-Daten muss eigenständig serialisiert bzw. beim Verbrauch deserialisiert werden.

## Nachrichten verbrauchen
Erstellen Sie die Datei `app/queue/stomp/MyMailSend.php` (beliebiger Klassenname, PSR-4-konform).
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // Warteschlangenname
    public $queue = 'examples';

    // Verbindungsname, entspricht der Verbindung in stomp.php
    public $connection = 'default';

    // Bei Wert 'client' muss $ack_resolver->ack() aufgerufen werden, um den Server über erfolgreiche Verarbeitung zu informieren
    // Bei Wert 'auto' ist kein Aufruf von $ack_resolver->ack() nötig
    public $ack = 'auto';

    // Verbrauch
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // Bei Array-Daten ist eine eigene Deserialisierung erforderlich
        var_export(json_decode($data, true)); // gibt ['to' => 'tom@gmail.com', 'content' => 'hello'] aus
        // Server über erfolgreiche Verarbeitung informieren
        $ack_resolver->ack(); // bei ack 'auto' kann dieser Aufruf entfallen
    }
}
```

# Stomp-Protokoll in RabbitMQ aktivieren
RabbitMQ aktiviert das Stomp-Protokoll standardmäßig nicht. Führen Sie folgenden Befehl zur Aktivierung aus:
```
rabbitmq-plugins enable rabbitmq_stomp
```
Nach der Aktivierung ist der Standard-Port für Stomp 61613.
