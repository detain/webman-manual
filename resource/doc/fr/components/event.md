# Gestion des événements
`webman/event` fournit un mécanisme d'événements sophistiqué qui permet d'exécuter une logique métier sans modifier le code, en découplant les modules. Scénario typique : lorsqu'un nouvel utilisateur s'inscrit, il suffit de publier un événement personnalisé comme `user.register`, et chaque module peut le recevoir et exécuter la logique correspondante.

## Installation
`composer require webman/event`

## S'abonner aux événements
Les abonnements se configurent de manière unifiée via le fichier `config/event.php`.

```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...autres fonctions de traitement...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...autres fonctions de traitement...
    ]
];
```

**Remarque :**
- `user.register`, `user.logout`, etc. sont des noms d'événements (chaînes). Il est recommandé d'utiliser des mots en minuscules séparés par des points (`.`).
- Un événement peut être traité par plusieurs fonctions ; elles sont appelées dans l'ordre de la configuration.

## Fonctions de traitement
Les fonctions de traitement peuvent être des méthodes de classe, des fonctions ou des closures.

Exemple : créez la classe `app/event/User.php` (créez le répertoire si nécessaire) :

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

## Publier un événement
Utilisez `Event::dispatch($event_name, $data);` ou `Event::emit($event_name, $data);` pour publier un événement. Par exemple :

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

Deux méthodes permettent de publier : `Event::dispatch($event_name, $data);` et `Event::emit($event_name, $data);` — les paramètres sont identiques. La différence : `emit` intercepte les exceptions en interne ; si une fonction lève une exception, les autres continuent. Avec `dispatch`, les exceptions ne sont pas interceptées ; si une fonction lève une exception, l'exécution s'arrête et l'exception est propagée.

> **Conseil**
> Le paramètre `$data` peut être n'importe quel type de données (tableau, instance de classe, chaîne, etc.).

## Écoute d'événements par joker
L'écoute par joker permet de gérer plusieurs événements avec un même listener. Par exemple dans `config/event.php` :

```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```

Le nom concret de l'événement est obtenu via le deuxième paramètre `$event_data` de la fonction de traitement :

```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // nom concret, ex. user.register, user.logout, etc.
        var_export($user);
    }
}
```

## Arrêter la transmission
Si une fonction de traitement retourne `false`, la transmission de l'événement s'arrête.

## Closures pour le traitement
La fonction de traitement peut être une méthode de classe ou une closure. Par exemple :

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

## Voir les événements et listeners
Utilisez la commande `php webman event:list` pour afficher tous les événements et listeners configurés.

## Périmètre de prise en charge
Outre le projet principal, les [plugins de base](../plugin/base.md) et les [plugins d'application](../app/app.md) prennent en charge la configuration `event.php`.
**Fichier de configuration plugin de base :** `config/plugin/éditeur/nom-plugin/event.php`
**Fichier de configuration plugin d'application :** `plugin/nom-plugin/config/event.php`

## Remarques
La gestion des événements n'est pas asynchrone. Elle n'est pas adaptée aux traitements lents ; ceux-ci doivent passer par des files de messages, par exemple [webman/redis-queue](https://www.workerman.net/plugin/12).
