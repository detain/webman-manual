# think-cache

think-cache è un componente estratto dal framework thinkphp, con supporto per il connection pool aggiunto. Supporta automaticamente ambienti sia con coroutine che senza coroutine.

## Installazione
`composer require -W webman/think-cache`

È necessario riavviare (restart) dopo l'installazione (reload non è efficace)

### File di configurazione

Il file di configurazione è `config/think-cache.php`

## Utilizzo

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
## API fornita
```php
// Impostare la cache
Cache::set('val','value',600);
// Verificare se la cache esiste
Cache::has('val');
// Ottenere la cache
Cache::get('val');
// Eliminare la cache
Cache::delete('val');
// Svuotare la cache
Cache::clear();
// Leggere e eliminare la cache
Cache::pull('val');
// Scrivere se non esiste
Cache::remember('val',10);

// Per dati cache numerici
// Incrementare la cache di 1
Cache::inc('val');
// Incrementare la cache di 5
Cache::inc('val',5);
// Decrementare la cache di 1
Cache::dec('val');
// Decrementare la cache di 5
Cache::dec('val',5);

// Utilizzare i tag della cache
Cache::tag('tag_name')->set('val','value',600);
// Eliminare la cache sotto un tag
Cache::tag('tag_name')->clear();
// Supporta tag multipli
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// Eliminare la cache sotto tag multipli
Cache::tag(['tag1','tag2'])->clear();

// Utilizzare diversi store di cache
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


