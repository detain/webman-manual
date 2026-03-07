# think-cache

think-cache ist eine aus dem thinkphp-Framework extrahierte Komponente mit Verbindungspool-Unterstützung und funktioniert automatisch in Koroutinen- und Nicht-Koroutinenumgebungen.

## Installation
`composer require -W webman/think-cache`

Nach der Installation ist ein Neustart (restart) erforderlich (reload ist wirkungslos)

### Konfigurationsdatei

Die Konfigurationsdatei ist `config/think-cache.php`

## Verwendung

  ```php
  <?php
  namespace app\controller;
    
  use support\Request;
  use support\think\Cache;
  
  class UserController
  {
      public function db(Request $request)
      {
          $key = 'test_key';
          Cache::set($key, rand());
          return response(Cache::get($key));
      }
  }
  ```
## Bereitgestellte API
```php
// Cache setzen
Cache::set('val','value',600);
// Prüfen, ob Cache existiert
Cache::has('val');
// Cache abrufen
Cache::get('val');
// Cache löschen
Cache::delete('val');
// Cache leeren
Cache::clear();
// Cache lesen und löschen
Cache::pull('val');
// Schreiben, wenn nicht vorhanden
Cache::remember('val',10);

// Für numerische Cache-Daten
// Cache um 1 erhöhen
Cache::inc('val');
// Cache um 5 erhöhen
Cache::inc('val',5);
// Cache um 1 verringern
Cache::dec('val');
// Cache um 5 verringern
Cache::dec('val',5);

// Cache-Tags verwenden
Cache::tag('tag_name')->set('val','value',600);
// Cache unter einem Tag löschen
Cache::tag('tag_name')->clear();
// Mehrere Tags werden unterstützt
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// Cache unter mehreren Tags löschen
Cache::tag(['tag1','tag2'])->clear();

// Verschiedene Cache-Stores verwenden
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


