# Gestion des sessions webman

## Exemple
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

Obtenez l'instance `Workerman\Protocols\Http\Session` via `$request->session();` et utilisez ses méthodes pour ajouter, modifier ou supprimer les données de session.

> **Remarque**
> Les données de session sont enregistrées automatiquement à la destruction de l'objet de session.
> Si l'objet de session est stocké dans une variable globale, il ne sera pas détruit et ne sera pas sauvegardé automatiquement. Il faut alors appeler manuellement `$session->save()` pour enregistrer les données.

## Récupérer toutes les données de session
```php
$session = $request->session();
$all = $session->all();
```
Retourne un tableau. Si aucune donnée de session n'existe, un tableau vide est retourné.


## Récupérer une valeur de session
```php
$session = $request->session();
$name = $session->get('name');
```
Retourne null si la donnée n'existe pas.

Vous pouvez passer une valeur par défaut comme second argument à la méthode `get`. Si la valeur correspondante n'est pas trouvée dans le tableau de session, la valeur par défaut est retournée. Par exemple :
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## Stocker des données de session
Utilisez la méthode `set` pour stocker une donnée.
```php
$session = $request->session();
$session->set('name', 'tom');
```
La méthode `set` ne retourne rien. Les données de session sont enregistrées automatiquement à la destruction de l'objet de session.

Utilisez la méthode `put` pour stocker plusieurs valeurs.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
De même, la méthode `put` ne retourne rien.

## Supprimer des données de session
Utilisez la méthode `forget` pour supprimer une ou plusieurs données de session.
```php
$session = $request->session();
// Supprimer une valeur
$session->forget('name');
// Supprimer plusieurs valeurs
$session->forget(['name', 'age']);
```

Le système fournit également la méthode `delete`. Contrairement à `forget`, elle ne peut supprimer qu'une seule valeur.
```php
$session = $request->session();
// Équivaut à $session->forget('name');
$session->delete('name');
```


## Récupérer et supprimer une valeur de session
```php
$session = $request->session();
$name = $session->pull('name');
```
Équivaut au code suivant :
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
Retourne null si la valeur de session correspondante n'existe pas.


## Supprimer toutes les données de session
```php
$request->session()->flush();
```
Ne retourne rien. Les données de session sont automatiquement supprimées du stockage à la destruction de l'objet de session.


## Vérifier si une valeur de session existe
```php
$session = $request->session();
$has = $session->has('name');
```
Retourne false si la valeur de session n'existe pas ou est null ; sinon retourne true.

```php
$session = $request->session();
$has = $session->exists('name');
```
Le code ci-dessus vérifie aussi l'existence d'une valeur de session. La différence : `exists` retourne true même lorsque la valeur est null.

## Fonction helper session()

webman fournit la fonction helper `session()` pour effectuer les mêmes opérations.
```php
// Obtenir l'instance de session
$session = session();
// Équivaut à
$session = $request->session();

// Obtenir une valeur
$value = session('key', 'default');
// Équivaut à
$value = session()->get('key', 'default');
// Équivaut à
$value = $request->session()->get('key', 'default');

// Assigner des valeurs à la session
session(['key1'=>'value1', 'key2' => 'value2']);
// Équivaut à
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// Équivaut à
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## Fichier de configuration
Le fichier de configuration de session se trouve dans `config/session.php`. Son contenu ressemble à :
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class ou RedisSessionHandler::class ou RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // Si handler est FileSessionHandler::class, la valeur est 'file',
    // si handler est RedisSessionHandler::class, la valeur est 'redis'
    // si handler est RedisClusterSessionHandler::class, la valeur est 'redis_cluster' (cluster Redis)
    'type'    => 'file',

    // Chaque handler utilise une configuration différente
    'config' => [
        // Configuration quand type est 'file'
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // Configuration quand type est 'redis'
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

    'session_name' => 'PHPSID', // Nom du cookie pour stocker le session_id
    'auto_update_timestamp' => false,  // Rafraîchir automatiquement la session, par défaut : désactivé
    'lifetime' => 7*24*60*60,          // Durée de vie de la session
    'cookie_lifetime' => 365*24*60*60, // Durée de vie du cookie stockant le session_id
    'cookie_path' => '/',              // Chemin du cookie stockant le session_id
    'domain' => '',                    // Domaine du cookie stockant le session_id
    'http_only' => true,               // Activer httpOnly, par défaut : activé
    'secure' => false,                 // Activer la session uniquement en HTTPS, par défaut : désactivé
    'same_site' => '',                 // Prévention CSRF et suivi utilisateur, valeurs possibles : strict/lax/none
    'gc_probability' => [1, 1000],     // Probabilité de nettoyage de la session
];
```

## Sécurité
Il n'est pas recommandé de stocker directement des instances de classes dans la session, notamment des instances provenant de sources non fiables. La désérialisation peut entraîner des risques de sécurité.

