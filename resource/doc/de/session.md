# webman Session-Verwaltung

## Beispiel
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

Über `$request->session();` erhalten Sie eine Instanz von `Workerman\Protocols\Http\Session` und können mit deren Methoden Sitzungsdaten hinzufügen, ändern oder löschen.

> **Hinweis**
> Sitzungsdaten werden automatisch gespeichert, wenn das Session-Objekt zerstört wird.
> Wird das Session-Objekt in einer globalen Variable gespeichert, verhindert dies die Zerstörung des Objekts und somit die automatische Speicherung. In diesem Fall müssen Sie manuell `$session->save()` aufrufen.

## Alle Sitzungsdaten abrufen
```php
$session = $request->session();
$all = $session->all();
```
Gibt ein Array zurück. Bei fehlenden Sitzungsdaten wird ein leeres Array zurückgegeben.


## Einen Wert aus der Sitzung abrufen
```php
$session = $request->session();
$name = $session->get('name');
```
Gibt null zurück, wenn die Daten nicht vorhanden sind.

Sie können der `get`-Methode auch einen Standardwert als zweiten Parameter übergeben. Wird der entsprechende Wert im Sitzungsarray nicht gefunden, wird dieser Standardwert zurückgegeben. Beispiel:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## Sitzungsdaten speichern
Verwenden Sie die `set`-Methode, um einen einzelnen Datensatz zu speichern.
```php
$session = $request->session();
$session->set('name', 'tom');
```
Die `set`-Methode gibt keinen Wert zurück. Die Sitzung wird automatisch gespeichert, wenn das Sitzungsobjekt zerstört wird.

Zum Speichern mehrerer Werte verwenden Sie die `put`-Methode.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
Auch die `put`-Methode gibt keinen Wert zurück.

## Sitzungsdaten löschen
Verwenden Sie die `forget`-Methode, um einen oder mehrere Sitzungswerte zu löschen.
```php
$session = $request->session();
// Ein Eintrag löschen
$session->forget('name');
// Mehrere Einträge löschen
$session->forget(['name', 'age']);
```

Darüber hinaus steht die Methode `delete` zur Verfügung, die im Gegensatz zu `forget` nur einen Eintrag löschen kann.
```php
$session = $request->session();
// Entspricht $session->forget('name');
$session->delete('name');
```


## Einen Sitzungswert abrufen und löschen
```php
$session = $request->session();
$name = $session->pull('name');
```
Entspricht folgendem Code:
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
Gibt null zurück, wenn der Sitzungswert nicht existiert.


## Alle Sitzungsdaten löschen
```php
$request->session()->flush();
```
Gibt keinen Wert zurück. Die Sitzungsdaten werden automatisch aus dem Speicher gelöscht, wenn das Sitzungsobjekt zerstört wird.


## Prüfen, ob ein Sitzungswert existiert
```php
$session = $request->session();
$has = $session->has('name');
```
Gibt false zurück, wenn der Sitzungswert nicht existiert oder null ist; andernfalls true.

```php
$session = $request->session();
$has = $session->exists('name');
```
Der obige Code prüft ebenfalls, ob ein Sitzungswert existiert. Der Unterschied: `exists` gibt auch dann true zurück, wenn der Sitzungswert null ist.

## Hilfsfunktion session()

webman stellt die Hilfsfunktion `session()` zur Verfügung, die dieselbe Funktionalität bietet.
```php
// Sitzungsinstanz abrufen
$session = session();
// Entspricht
$session = $request->session();

// Einen Wert abrufen
$value = session('key', 'default');
// Entspricht
$value = session()->get('key', 'default');
// Entspricht
$value = $request->session()->get('key', 'default');

// Werte der Sitzung zuweisen
session(['key1'=>'value1', 'key2' => 'value2']);
// Entspricht
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// Entspricht
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## Konfigurationsdatei
Die Sitzungskonfiguration befindet sich in `config/session.php`. Der Inhalt sieht in etwa so aus:
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class oder RedisSessionHandler::class oder RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // Bei FileSessionHandler::class ist der Wert 'file',
    // bei RedisSessionHandler::class ist er 'redis',
    // bei RedisClusterSessionHandler::class ist er 'redis_cluster' (Redis-Cluster)
    'type'    => 'file',

    // Verschiedene Handler verwenden verschiedene Konfigurationen
    'config' => [
        // Konfiguration bei type 'file'
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // Konfiguration bei type 'redis'
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // Cookie-Name zum Speichern der session_id
    'auto_update_timestamp' => false,  // Sitzung automatisch aktualisieren, Standard: aus
    'lifetime' => 7*24*60*60,          // Sitzungsablaufzeit
    'cookie_lifetime' => 365*24*60*60, // Ablaufzeit des Cookies für session_id
    'cookie_path' => '/',              // Cookie-Pfad für session_id
    'domain' => '',                    // Cookie-Domain für session_id
    'http_only' => true,               // httpOnly aktivieren, Standard: an
    'secure' => false,                 // Sitzung nur über HTTPS, Standard: aus
    'same_site' => '',                 // Schutz vor CSRF-Angriffen und Nutzerverfolgung, Optionen: strict/lax/none
    'gc_probability' => [1, 1000],     // Wahrscheinlichkeit der Sitzungsbereinigung
];
```

## Sicherheit
Es wird nicht empfohlen, Klasseninstanzen direkt in der Sitzung zu speichern, insbesondere von Klassen aus nicht vertrauenswürdigen Quellen. Bei der Deserialisierung können Sicherheitsrisiken entstehen.

