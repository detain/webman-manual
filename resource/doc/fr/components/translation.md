# Multilingue

La fonctionnalité multilingue utilise le composant [symfony/translation](https://github.com/symfony/translation).

## Installation
```
composer require symfony/translation
```

## Créer des paquets de langue
webman stocke les paquets de langue par défaut dans le répertoire `resource/translations` (à créer si nécessaire). Pour changer le répertoire, configurez-le dans `config/translation.php`.
Chaque langue correspond à un sous-dossier, et les définitions sont stockées dans `messages.php` par défaut. Exemple :
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

Tous les fichiers de langue retournent un tableau, par exemple :
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## Configuration

`config/translation.php`

```php
return [
    // Langue par défaut
    'locale' => 'zh_CN',
    // Langue de repli : si une traduction n'est pas trouvée dans la langue actuelle, celle de la langue de repli est utilisée
    'fallback_locale' => ['zh_CN', 'en'],
    // Répertoire où sont stockés les fichiers de langue
    'path' => base_path() . '/resource/translations',
];
```

## Traduction

Utilisez la méthode `trans()` pour la traduction.

Créer le fichier de langue `resource/translations/zh_CN/messages.php` :
```php
return [
    'hello' => '你好 世界!',
];
```

Créer le fichier `app/controller/UserController.php` :
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        $hello = trans('hello'); // 你好 世界!
        return response($hello);
    }
}
```

L'accès à `http://127.0.0.1:8787/user/get` retournera "你好 世界!"

## Changer la langue par défaut

Utilisez la méthode `locale()` pour changer de langue.

Ajouter le fichier de langue `resource/translations/en/messages.php` :
```php
return [
    'hello' => 'hello world!',
];
```

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // Changer de langue
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
L'accès à `http://127.0.0.1:8787/user/get` retournera "hello world!"

Vous pouvez aussi utiliser le 4ᵉ paramètre de la fonction `trans()` pour changer temporairement de langue. Par exemple, l'exemple ci-dessus est équivalent à :
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // Le 4ᵉ paramètre change la langue
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## Définir la langue explicitement pour chaque requête
translation est un singleton : toutes les requêtes partagent la même instance. Si une requête définit la langue par défaut avec `locale()`, cela affectera toutes les requêtes suivantes du processus. Il faut donc définir la langue explicitement pour chaque requête, par exemple avec le middleware suivant :

Créer le fichier `app/middleware/Lang.php` (créer le répertoire si nécessaire) :
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Lang implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        locale(session('lang', 'zh_CN'));
        return $handler($request);
    }
}
```

Ajouter le middleware global dans `config/middleware.php` :
```php
return [
    // Middleware global
    '' => [
        // ... autres middlewares omis
        app\middleware\Lang::class,
    ]
];
```


## Utiliser des espaces réservés
Parfois un message contient des variables à traduire, par exemple
```php
trans('hello ' . $name);
```
Dans ce cas, on utilise des espaces réservés.

Modifier `resource/translations/zh_CN/messages.php` :
```php
return [
    'hello' => '你好 %name%!',
];
```
Lors de la traduction, les valeurs sont passées via le 2ᵉ paramètre :
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## Gérer les pluriels
Dans certaines langues, la structure de la phrase varie selon la quantité. Par exemple, « There is %count% apple » est correct quand `%count%` vaut 1, mais incorrect sinon.

Dans ce cas, on utilise le **pipe** (`|`) pour lister les formes plurielles.

Ajouter `apple_count` dans `resource/translations/en/messages.php` :
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

On peut aussi spécifier des plages numériques pour des règles de pluriel plus complexes :
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## Spécifier le fichier de langue

Par défaut le fichier s'appelle `messages.php`, mais d'autres noms sont possibles.

Créer le fichier `resource/translations/zh_CN/admin.php` :
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

Spécifier le fichier avec le 3ᵉ paramètre de `trans()` (sans l'extension `.php`) :
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## Plus d'informations
Voir la [documentation symfony/translation](https://symfony.com/doc/current/translation.html)
