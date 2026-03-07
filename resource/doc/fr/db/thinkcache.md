# think-cache

think-cache est un composant extrait du framework thinkphp, auquel a été ajouté un support de pool de connexions. Il prend automatiquement en charge les environnements avec et sans coroutines.

## Installation
`composer require -W webman/think-cache`

Un redémarrage (restart) est requis après l'installation (reload est inefficace)

### Fichier de configuration

Le fichier de configuration est `config/think-cache.php`

## Utilisation

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
## API fournie
```php
// Définir un cache
Cache::set('val','value',600);
// Vérifier si le cache existe
Cache::has('val');
// Récupérer le cache
Cache::get('val');
// Supprimer le cache
Cache::delete('val');
// Vider le cache
Cache::clear();
// Lire et supprimer le cache
Cache::pull('val');
// Écrire si inexistant
Cache::remember('val',10);

// Pour les données de cache numériques
// Incrémenter le cache de 1
Cache::inc('val');
// Incrémenter le cache de 5
Cache::inc('val',5);
// Décrémenter le cache de 1
Cache::dec('val');
// Décrémenter le cache de 5
Cache::dec('val',5);

// Utiliser les étiquettes de cache
Cache::tag('tag_name')->set('val','value',600);
// Supprimer le cache sous une étiquette
Cache::tag('tag_name')->clear();
// Prend en charge plusieurs étiquettes
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// Supprimer le cache sous plusieurs étiquettes
Cache::tag(['tag1','tag2'])->clear();

// Utiliser différents magasins de cache
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


