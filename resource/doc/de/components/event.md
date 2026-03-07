# Event-Eventverarbeitung
`webman/event` bietet einen raffinierten Ereignismechanismus, der es ermöglicht, Geschäftslogiken auszuführen, ohne den Code zu ändern, und damit die Entkopplung zwischen Geschäftsmodulen zu erreichen. Typisches Szenario: Wenn ein neuer Benutzer sich erfolgreich registriert, wird einfach ein benutzerdefiniertes Ereignis wie `user.register` veröffentlicht, und jedes Modul kann dieses Ereignis empfangen und die entsprechende Geschäftslogik ausführen.

## Installation
`composer require webman/event`

## Ereignis abonnieren
Ereignis-Abonnements werden einheitlich in der Datei `config/event.php` konfiguriert.

```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...andere Ereignisverarbeitungsfunktionen...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...andere Ereignisverarbeitungsfunktionen...
    ]
];
```

**Hinweis:**
- `user.register`, `user.logout` usw. sind Ereignisnamen (Zeichenketten). Empfohlen werden Kleinbuchstaben, mit Punkt (`.`) getrennt.
- Ein Ereignis kann mehreren Ereignisverarbeitungsfunktionen entsprechen; sie werden in der Reihenfolge der Konfiguration aufgerufen.

## Ereignisverarbeitungsfunktionen
Ereignisverarbeitungsfunktionen können beliebige Klassenmethoden, Funktionen oder Closure-Funktionen sein.

Erstellen Sie z. B. die Ereignisverarbeitungsklasse `app/event/User.php` (Verzeichnis bei Bedarf anlegen).

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

## Ereignis veröffentlichen
Verwenden Sie `Event::dispatch($event_name, $data);` oder `Event::emit($event_name, $data);` zum Veröffentlichen von Ereignissen, z. B.:

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

Es gibt zwei Funktionen zum Veröffentlichen: `Event::dispatch($event_name, $data);` und `Event::emit($event_name, $data);` — beide mit denselben Parametern. Der Unterschied: `emit` fängt Ausnahmen intern ab; wirft eine Verarbeitungsfunktion eine Ausnahme, laufen die anderen dennoch weiter. Bei `dispatch` werden Ausnahmen nicht abgefangen; wirft eine Verarbeitungsfunktion eine Ausnahme, bricht die Ausführung ab und die Ausnahme wird weitergeworfen.

> **Hinweis**
> Der Parameter `$data` kann beliebige Daten sein (z. B. Arrays, Klasseninstanzen, Zeichenketten).

## Wildcard-Ereignisüberwachung
Mit Wildcard-Registrierung können Sie mehrere Ereignisse mit demselben Listener behandeln, z. B. in `config/event.php`:

```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```

Über den zweiten Parameter `$event_data` der Ereignisverarbeitungsfunktion erhalten Sie den konkreten Ereignisnamen:

```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // konkreter Ereignisname, z. B. user.register, user.logout usw.
        var_export($user);
    }
}
```

## Ereignisübertragung stoppen
Wenn die Ereignisverarbeitungsfunktion `false` zurückgibt, wird die Übertragung des Ereignisses gestoppt.

## Ereignisverarbeitung mit Closure
Die Ereignisverarbeitungsfunktion kann eine Klassenmethode oder eine Closure sein, z. B.:

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

## Ereignisse und Listener anzeigen
Mit dem Befehl `php webman event:list` können Sie alle im Projekt konfigurierten Ereignisse und Listener anzeigen.

## Unterstützte Bereiche
Neben dem Hauptprojekt unterstützen auch [Basis-Plugins](../plugin/base.md) und [App-Plugins](../app/app.md) die `event.php`-Konfiguration.
**Basis-Plugin-Konfiguration:** `config/plugin/Hersteller/Pluginname/event.php`
**App-Plugin-Konfiguration:** `plugin/Pluginname/config/event.php`

## Hinweise
Die Event-Verarbeitung ist nicht asynchron und nicht für langsame Geschäftslogik geeignet; diese sollte über Message Queues abgearbeitet werden, z. B. [webman/redis-queue](https://www.workerman.net/plugin/12).
