# Injection automatique de dépendances

Dans webman, l'injection automatique de dépendances est une fonctionnalité optionnelle, désactivée par défaut. Si vous en avez besoin, il est recommandé d'utiliser [php-di](https://php-di.org/doc/getting-started.html). Voici comment utiliser `php-di` avec webman.

## Installation

```
composer require php-di/php-di:^7.0
```

Modifiez la configuration `config/container.php`. Le contenu final doit être le suivant :

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> Le fichier `config/container.php` doit finalement retourner une instance de conteneur conforme à la spécification PSR-11. Si vous ne souhaitez pas utiliser `php-di`, vous pouvez créer et retourner une autre instance de conteneur conforme PSR-11 ici. La configuration par défaut fournit uniquement les fonctionnalités de base du conteneur webman.

## Injection par constructeur

Créez le fichier `app/service/Mailer.php` (créez le répertoire s'il n'existe pas) avec le contenu suivant :

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // Code d'envoi du courrier électronique omis
    }
}
```

Le contenu de `app/controller/UserController.php` est le suivant :

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{

    public function __construct(private Mailer $mailer)
    {
    }

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Bonjour et bienvenue !');
        return response('ok');
    }
}
```

En conditions normales, le code suivant serait nécessaire pour instancier `app\controller\UserController` :

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

En utilisant `php-di`, le développeur n'a pas besoin d'instancier manuellement `Mailer` dans le contrôleur — webman s'en charge automatiquement. S'il existe d'autres dépendances lors de l'instanciation de `Mailer`, webman les instanciera et les injectera également. Aucun travail d'initialisation n'est requis du développeur.

> **Remarque**
> Seules les instances créées par le framework ou `php-di` permettent l'injection automatique de dépendances. Les instances créées manuellement avec `new` ne le peuvent pas. Pour injecter, utilisez l'interface `support\Container` à la place de `new`, par exemple :

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// Les instances créées avec new ne permettent pas l'injection de dépendances
$user_service = new UserService;
// Les instances créées avec new ne permettent pas l'injection de dépendances
$log_service = new LogService($path, $name);

// Les instances créées avec Container permettent l'injection de dépendances
$user_service = Container::get(UserService::class);
// Les instances créées avec Container permettent l'injection de dépendances
$log_service = Container::make(LogService::class, [$path, $name]);
```

## Injection par attribut

En plus de l'injection par constructeur, vous pouvez utiliser l'injection par attribut. En poursuivant l'exemple précédent, modifiez `app\controller\UserController` comme suit :

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private Mailer $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Bonjour et bienvenue !');
        return response('ok');
    }
}
```

Cet exemple utilise l'attribut `#[Inject]` pour l'injection et injecte automatiquement l'instance dans le membre selon le type d'objet. L'effet est identique à l'injection par constructeur, mais le code est plus concis.

> **Remarque**
> webman ne prend pas en charge l'injection de paramètres de contrôleur avant la version 1.4.6. Par exemple, le code suivant n'est pas pris en charge lorsque webman<=1.4.6 :

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // L'injection de paramètres de contrôleur n'est pas prise en charge avant la version 1.4.6
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'Bonjour et bienvenue !');
        return response('ok');
    }
}
```

## Injection personnalisée par constructeur

Parfois les paramètres passés au constructeur ne sont pas des instances de classe mais des chaînes, des nombres, des tableaux ou d'autres données non objets. Par exemple, le constructeur de Mailer peut avoir besoin de l'IP et du port du serveur SMTP :

```php
<?php
namespace app\service;

class Mailer
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // Code d'envoi du courrier électronique omis
    }
}
```

Ce cas ne permet pas d'utiliser directement l'injection automatique par constructeur, car `php-di` ne peut pas déterminer les valeurs de `$smtp_host` et `$smtp_port`. Dans ce cas, utilisez l'injection personnalisée.

Ajoutez le code suivant dans `config/dependence.php` (créez le fichier s'il n'existe pas) :

```php
return [
    // ... Autres configurations omises

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

Lorsque l'injection de dépendances a besoin d'obtenir une instance de `app\service\Mailer`, elle utilisera automatiquement l'instance créée dans cette configuration.

Notez que `config/dependence.php` utilise `new` pour instancier la classe `Mailer`. Cela ne pose pas de problème dans cet exemple, mais si la classe `Mailer` dépend d'autres classes ou utilise l'injection par attribut en interne, l'initialisation avec `new` n'effectuera pas l'injection automatique de dépendances. La solution est d'utiliser l'injection personnalisée par interface et d'initialiser les classes via `Container::get(nom de classe)` ou `Container::make(nom de classe, [paramètres du constructeur])`.

## Injection personnalisée par interface

Dans les projets réels, il est préférable de programmer contre des interfaces plutôt que des classes concrètes. Par exemple, `app\controller\UserController` devrait dépendre de `app\service\MailerInterface` plutôt que de `app\service\Mailer`.

Définissez l'interface `MailerInterface` :

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

Définissez l'implémentation de `MailerInterface` :

```php
<?php
namespace app\service;

class Mailer implements MailerInterface
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // Code d'envoi du courrier électronique omis
    }
}
```

Utilisez `MailerInterface` au lieu de l'implémentation concrète :

```php
<?php
namespace app\controller;

use support\Request;
use app\service\MailerInterface;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private MailerInterface $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Bonjour et bienvenue !');
        return response('ok');
    }
}
```

Définissez l'implémentation de `MailerInterface` dans `config/dependence.php` :

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

Lorsque l'application a besoin d'utiliser l'interface `MailerInterface`, l'implémentation `Mailer` sera automatiquement utilisée.

> L'avantage de programmer contre des interfaces est qu'en cas de remplacement d'un composant, il n'est pas nécessaire de modifier le code métier — uniquement l'implémentation concrète dans `config/dependence.php`. C'est également très utile pour les tests unitaires.

## Autres injections personnalisées

Outre les dépendances de classes, `config/dependence.php` peut définir d'autres valeurs comme des chaînes, des nombres, des tableaux, etc.

Par exemple, si `config/dependence.php` est défini comme suit :

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

Vous pouvez injecter `smtp_host` et `smtp_port` dans les propriétés de classe avec `#[Inject]` :

```php
<?php
namespace app\service;

use DI\Attribute\Inject;

class Mailer
{
    #[Inject("smtp_host")]
    private $smtpHost;

    #[Inject("smtp_port")]
    private $smtpPort;

    public function mail($email, $content)
    {
        // Code d'envoi du courrier électronique omis
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // Affichera 192.168.1.11:25
    }
}
```

# Chargement paresseux (Lazy Loading)

> Le chargement paresseux est un modèle de conception qui diffère la création ou l'initialisation d'objets jusqu'à ce qu'ils soient réellement nécessaires.

Cette fonctionnalité nécessite une dépendance supplémentaire. Le paquet suivant est un fork de `ocramius/proxy-manager` ; le dépôt original ne prend pas en charge PHP 8.

```
composer require friendsofphp/proxy-manager-lts
```

Utilisation :

```php
<?php

use DI\Attribute\Injectable;
use DI\Attribute\Inject;

#[Injectable(lazy: true)]
class MyClass
{
    private string $name;

    public function __construct()
    {
        echo "MyClass instancié\n";
        $this->name = "Lazy Loaded Object";
    }

    public function getName(): string
    {
        return $this->name;
    }
}

class Controller
{
    #[Inject]
    public MyClass $myClass;

    public function getClass()
    {
        echo "Nom de la classe proxy : " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

Sortie :

```
Nom de la classe proxy : ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass instancié
name: Lazy Loaded Object
```

Cet exemple montre que pour une classe déclarée avec l'attribut `#[Injectable]`, lorsqu'elle est injectée, une classe proxy est d'abord créée. La classe réelle n'est instanciée qu'au moment où l'une de ses méthodes est appelée.

# Dépendances circulaires

Les dépendances circulaires surviennent lorsque plusieurs classes dépendent les unes des autres, formant une boucle de dépendance fermée.

- Dépendance circulaire directe
  - Le module A dépend du module B et le module B dépend du module A
  - Forme la boucle : A → B → A

- Dépendance circulaire indirecte
  - Implique plusieurs modules dans un cycle de dépendance
  - Par exemple : A → B → C → A

Lors de l'utilisation de l'injection par attribut, `php-di` détecte automatiquement les dépendances circulaires et lève une exception. Si nécessaire, utilisez l'approche suivante :

```php
class userController
{

    // Supprimez ce code
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## Pour en savoir plus

Veuillez consulter la [documentation php-di](https://php-di.org/doc/getting-started.html).
