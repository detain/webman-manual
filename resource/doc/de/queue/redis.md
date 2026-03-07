# Redis-Warteschlange

Eine Nachrichtenwarteschlange basierend auf Redis, unterstützt die verzögerte Verarbeitung von Nachrichten.

## Installation
`composer require webman/redis-queue`

## Konfigurationsdatei
Die Redis-Konfigurationsdatei wird automatisch unter `{Hauptprojekt}/config/plugin/webman/redis-queue/redis.php` erstellt, mit Inhalt ähnlich folgendem:
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // Passwort, optional
            'db' => 0,            // Datenbank
            'max_attempts'  => 5, // Wiederholungsversuche nach fehlgeschlagener Verarbeitung
            'retry_seconds' => 5, // Wiederholungsintervall in Sekunden
        ]
    ],
];
```

### Wiederholung bei fehlgeschlagener Verarbeitung
Wenn die Verarbeitung fehlschlägt (Ausnahme tritt auf), wird die Nachricht in die verzögerte Warteschlange gelegt und wartet auf den nächsten Wiederholungsversuch. Die Anzahl der Wiederholungen wird durch `max_attempts` gesteuert, das Intervall gemeinsam durch `retry_seconds` und `max_attempts`. Z.B. bei `max_attempts` = 5 und `retry_seconds` = 10 beträgt das Intervall für den 1. Versuch `1*10` Sekunden, für den 2. `2*10` Sekunden, für den 3. `3*10` Sekunden usw. bis zu 5 Versuchen. Überschreitet die Wiederholungsanzahl die `max_attempts`-Einstellung, landet die Nachricht in der Fehlschlange mit Schlüssel `{redis-queue}-failed`.

## Nachrichtenversand (synchron)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // Warteschlangenname
        $queue = 'send-mail';
        // Daten, können direkt als Array übergeben werden, keine Serialisierung nötig
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Nachricht senden
        Redis::send($queue, $data);
        // Verzögerte Nachricht senden, wird nach 60 Sekunden verarbeitet
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
Bei erfolgreichem Versand gibt `Redis::send()` true zurück, sonst false oder wirft eine Ausnahme.

> **Tipp**
> Es kann zu Abweichungen in der Verarbeitungszeit der verzögerten Warteschlange kommen. Z.B. wenn die Verarbeitungsgeschwindigkeit langsamer ist als die Produktionsgeschwindigkeit, kann sich die Warteschlange stauen und Verzögerungen entstehen. Abhilfe: mehr Verarbeitungsprozesse starten.

## Nachrichtenversand (asynchron)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // Warteschlangenname
        $queue = 'send-mail';
        // Daten, können direkt als Array übergeben werden, keine Serialisierung nötig
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Nachricht senden
        Client::send($queue, $data);
        // Verzögerte Nachricht senden, wird nach 60 Sekunden verarbeitet
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` hat keinen Rückgabewert. Es ist ein asynchroner Push und garantiert keine 100%ige Zustellung an Redis.

> **Tipp**
> Das Prinzip von `Client::send()` ist, lokal eine Speicher-Warteschlange anzulegen und Nachrichten asynchron mit Redis zu synchronisieren (Synchronisation ist schnell, etwa 10.000 Nachrichten pro Sekunde). Wird der Prozess neu gestartet, bevor Daten der lokalen Speicher-Warteschlange vollständig synchronisiert sind, können Nachrichten verloren gehen. Asynchroner Versand mit `Client::send()` eignet sich für unwichtige Nachrichten.

> **Tipp**
> `Client::send()` ist asynchron und kann nur in der Workerman-Laufzeitumgebung verwendet werden. Für Kommandozeilen-Skripte die synchrone Schnittstelle `Redis::send()` verwenden.

## Nachrichtenversand aus anderen Projekten
Manchmal müssen Sie Nachrichten aus anderen Projekten versenden und können `webman\redis-queue` nicht nutzen. In solchen Fällen können Sie die folgende Funktion verwenden, um Nachrichten in die Warteschlange zu senden.

```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting';
    $queue_delay = '{redis-queue}-delayed';
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```

Dabei ist der Parameter `$redis` die Redis-Instanz. Z.B. die Nutzung der Redis-Extension sieht so aus:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## Verarbeitung
Die Konfigurationsdatei des Verarbeitungsprozesses liegt unter `{Hauptprojekt}/config/plugin/webman/redis-queue/process.php`. Das Verzeichnis der Verbraucher ist unter `{Hauptprojekt}/app/queue/redis/`.

Der Befehl `php webman redis-queue:consumer my-send-mail` erzeugt die Datei `{Hauptprojekt}/app/queue/redis/MyMailSend.php`.

> **Tipp**
> Dieser Befehl erfordert die Installation des [Konsolen-Plugins](../plugin/console.md). Ohne Installation können Sie manuell eine ähnliche Datei anlegen:

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // Zu verarbeitender Warteschlangenname
    public $queue = 'send-mail';

    // Verbindungsname, entspricht der Verbindung in plugin/webman/redis-queue/redis.php
    public $connection = 'default';

    // Verarbeitung
    public function consume($data)
    {
        // Keine Deserialisierung nötig
        var_export($data); // Gibt ['to' => 'tom@gmail.com', 'content' => 'hello'] aus
    }
    // Callback bei Verarbeitungsfehler
    /* 
    $package = [
        'id' => 1357277951, // Nachrichten-ID
        'time' => 1709170510, // Nachrichtenzeit
        'delay' => 0, // Verzögerungszeit
        'attempts' => 2, // Verarbeitungsanzahl
        'queue' => 'send-mail', // Warteschlangenname
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // Nachrichteninhalt
        'max_attempts' => 5, // Max. Wiederholungen
        'error' => 'Fehlermeldung' // Fehlermeldung
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // Keine Deserialisierung nötig
        var_export($package); 
    }
}
```

> **Hinweis**
> Verarbeitung gilt als erfolgreich, wenn währenddessen keine Ausnahme oder kein Error geworfen wird; sonst Fehlschlag und die Nachricht geht in die Wiederholungswarteschlange. redis-queue hat keinen ACK-Mechanismus; Sie können es als automatisches ACK betrachten (wenn keine Ausnahme oder kein Error auftritt). Um die aktuelle Nachricht als nicht erfolgreich verarbeitet zu markieren, können Sie manuell eine Ausnahme werfen, damit die Nachricht in die Wiederholungswarteschlange kommt. In der Praxis entspricht das einem ACK-Mechanismus.

> **Tipp**
> Verbraucher unterstützen mehrere Server und Prozesse, und dieselbe Nachricht wird **nicht** doppelt verarbeitet. Verarbeitete Nachrichten werden automatisch aus der Warteschlange entfernt; keine manuelle Löschung nötig.

> **Tipp**
> Verarbeitungsprozesse können mehrere verschiedene Warteschlangen gleichzeitig verarbeiten. Neue Warteschlangen erfordern keine Änderung der Konfiguration in `process.php`. Bei neuen Warteschlangen-Verbrauchern fügen Sie einfach die entsprechende `Consumer`-Klasse unter `app/queue/redis` hinzu und legen mit der Klassen-Eigenschaft `$queue` den zu verarbeitenden Warteschlangennamen fest.

> **Tipp**
> Windows-Benutzer müssen `php windows.php` ausführen, um webman zu starten, sonst wird der Verarbeitungsprozess nicht gestartet.

> **Tipp**
> Der onConsumeFailure-Callback wird bei jedem Verarbeitungsfehler ausgelöst. Dort können Sie die Logik nach einem Fehlschlag implementieren. (Diese Funktion erfordert `webman/redis-queue>=1.3.2` und `workerman/redis-queue>=1.2.1`)

## Unterschiedliche Verarbeitungsprozesse für verschiedene Warteschlangen
Standardmäßig teilen alle Verbraucher denselben Verarbeitungsprozess. Gelegentlich soll die Verarbeitung einiger Warteschlangen getrennt werden—z.B. langsame Geschäftsvorgänge in einer Prozessgruppe, schnelle in einer anderen. Dazu können die Verbraucher in zwei Verzeichnisse aufgeteilt werden, z.B. `app_path() . '/queue/redis/fast'` und `app_path() . '/queue/redis/slow'` (der Namespace der Verbraucher-Klasse muss entsprechend angepasst werden). Die Konfiguration sieht so aus:
```php
return [
    ...andere Konfigurationen ausgelassen...
    
    'redis_consumer_fast'  => [ // Schlüssel ist frei wählbar, kein Formatzwang, hier redis_consumer_fast genannt
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Verzeichnis der Verbraucher-Klassen
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // Schlüssel ist frei wählbar, kein Formatzwang, hier redis_consumer_slow genannt
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Verzeichnis der Verbraucher-Klassen
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

Damit liegen schnelle Verbraucher im Verzeichnis `queue/redis/fast` und langsame in `queue/redis/slow`, sodass den Warteschlangen gezielt Verarbeitungsprozesse zugewiesen werden können.

## Mehrfache Redis-Konfiguration
#### Konfiguration
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // Passwort, Typ String, optional
            'db' => 0,            // Datenbank
            'max_attempts'  => 5, // Wiederholungen nach Verarbeitungsfehler
            'retry_seconds' => 5, // Wiederholungsintervall in Sekunden
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // Passwort, Typ String, optional
            'db' => 0,            // Datenbank
            'max_attempts'  => 5, // Wiederholungen nach Verarbeitungsfehler
            'retry_seconds' => 5, // Wiederholungsintervall in Sekunden
        ]
    ],
];
```

Beachten Sie: Die Konfiguration enthält zusätzlich eine Redis-Konfiguration mit Schlüssel `other`.

#### Nachrichtenversand an mehrere Redis

```php
// Nachricht an Warteschlange mit Schlüssel `default` senden
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// Gleichbedeutend mit
Client::send($queue, $data);
Redis::send($queue, $data);

// Nachricht an Warteschlange mit Schlüssel `other` senden
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### Verarbeitung von mehreren Redis
Nachrichten von der Warteschlange mit Schlüssel `other` in der Konfiguration verarbeiten:
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // Zu verarbeitender Warteschlangenname
    public $queue = 'send-mail';

    // === Hier auf 'other' setzen, um von der Warteschlange mit Schlüssel 'other' in der Konfiguration zu verarbeiten ===
    public $connection = 'other';

    // Verarbeitung
    public function consume($data)
    {
        // Keine Deserialisierung nötig
        var_export($data);
    }
}
```

## Häufige Fragen

**Warum kommt der Fehler `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)` vor?**

Dieser Fehler tritt nur bei der asynchronen Schnittstelle `Client::send()` auf. Asynchroner Versand speichert Nachrichten zunächst im lokalen Speicher und sendet sie bei Prozess-Leerlauf an Redis. Ist die Empfangsgeschwindigkeit von Redis geringer als die Produktionsgeschwindigkeit oder ist der Prozess mit anderen Aufgaben beschäftigt und kommt nicht dazu, Nachrichten aus dem Speicher mit Redis zu synchronisieren, kann sich ein Rückstau bilden. Hält der Rückstau länger als 600 Sekunden an, wird dieser Fehler ausgelöst.

Lösung: Verwenden Sie die synchrone Schnittstelle `Redis::send()` für den Nachrichtenversand.
