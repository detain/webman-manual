# Processus de création et de publication d’un plugin de base

## Principe
1. Prenons l’exemple d’un plugin cross-origin : il est composé de trois éléments : un fichier de middleware, un fichier de config `middleware.php`, et un `Install.php` généré automatiquement par commande.
2. On utilise une commande pour empaqueter ces trois fichiers et les publier sur Composer.
3. Quand l’utilisateur installe le plugin cross-origin via Composer, `Install.php` copie le fichier middleware et la config vers `{projet principal}/config/plugin` pour que webman puisse les charger, et active ainsi la config cross-origin.
4. Quand l’utilisateur désinstalle le plugin via Composer, `Install.php` supprime les fichiers middleware et de config correspondants, ce qui désinstalle proprement le plugin.

## Spécifications
1. Un nom de plugin est composé de `vendor` et `nom du plugin`, ex. `webman/push`, ce qui correspond au nom du paquet Composer.
2. Les fichiers de config du plugin sont placés sous `config/plugin/vendor/nom du plugin/` (la commande console crée le répertoire de config automatiquement). Si le plugin n’a pas besoin de config, supprimez le répertoire créé automatiquement.
3. Le répertoire de config du plugin ne supporte que : `app.php` (config principale), `bootstrap.php` (démarrage des processus), `route.php` (routes), `middleware.php` (middlewares), `process.php` (processus personnalisés), `database.php` (BDD), `redis.php` (Redis), `thinkorm.php` (thinkorm). Ces fichiers sont reconnus automatiquement par webman.
4. Accès à la config : `config('plugin.vendor.nom du plugin.fichier de config.clé spécifique');`, ex. `config('plugin.webman.push.app.app_key')`.
5. Si le plugin a sa propre config BDD : pour `illuminate/database` utiliser `Db::connection('plugin.vendor.nom du plugin.connexion')`, pour `thinkorm` utiliser `Db::connect('plugin.vendor.nom du plugin.connexion')`.
6. Si un plugin place des fichiers métier dans `app/`, il faut éviter tout conflit avec le projet principal et les autres plugins.
7. Les plugins doivent éviter de copier des fichiers ou répertoires dans le projet principal. Par ex., pour le plugin cross-origin, seul le fichier de config est copié ; les fichiers middleware restent dans `vendor/webman/cros/src`.
8. Il est recommandé d’utiliser PascalCase pour les namespaces des plugins, ex. `Webman/Console`.

## Exemple

**Installer la ligne de commande `webman/console`**

`composer require webman/console`

### Créer un plugin

Supposons que le plugin à créer s’appelle `foo/admin` (nom du projet Composer à publier, en minuscules). Exécuter :

`php webman plugin:create --name=foo/admin`

Cela crée `vendor/foo/admin` pour les fichiers du plugin et `config/plugin/foo/admin` pour la config.

> Note
> `config/plugin/foo/admin` accepte : `app.php`, `bootstrap.php`, `route.php`, `middleware.php`, `process.php`, `database.php`, `redis.php`, `thinkorm.php`. Format identique à webman, fusion automatique.
> Accès avec le préfixe `plugin`, ex. `config('plugin.foo.admin.app')`.


### Exporter le plugin

Une fois le développement terminé, exécuter :

`php webman plugin:export --name=foo/admin`

Exporter

> Explication
> L’export copie `config/plugin/foo/admin` vers `vendor/foo/admin/src` et génère `Install.php`, utilisé à l’installation et à la désinstallation.
> Installation par défaut : copie la config de `vendor/foo/admin/src` vers `config/plugin` du projet.
> Désinstallation par défaut : suppression des fichiers de config du plugin sous `config/plugin` du projet.
> Vous pouvez modifier `Install.php` pour ajouter une logique personnalisée à l’installation et à la désinstallation.

### Soumettre le plugin
* Vous disposez déjà d’un compte [GitHub](https://github.com) et [Packagist](https://packagist.org).
* Créez un dépôt `admin` sur [GitHub](https://github.com) et poussez le code, ex. `https://github.com/votre-username/admin`.
* Allez sur `https://github.com/votre-username/admin/releases/new` et créez une release, ex. `v1.0.0`.
* Sur [Packagist](https://packagist.org), cliquez sur `Submit` et soumettez `https://github.com/votre-username/admin` pour publier le plugin.

> **Conseil**
> En cas de conflit de nom sur Packagist, choisir un autre nom de vendor, ex. remplacer `foo/admin` par `myfoo/admin`.

Lors des mises à jour : pousser le code sur GitHub, créer une nouvelle release sur `https://github.com/votre-username/admin/releases/new`, puis cliquer sur `Update` sur `https://packagist.org/packages/foo/admin`.

## Ajouter des commandes au plugin
Certains plugins nécessitent des commandes personnalisées. Par ex., après installation de `webman/redis-queue`, le projet dispose de la commande `redis-queue:consumer`. Exécuter `php webman redis-queue:consumer send-mail` génère rapidement une classe `SendMail.php`, ce qui accélère le développement.

Pour ajouter la commande `foo-admin:add` au plugin `foo/admin` :

### Créer une commande

**Créer le fichier `vendor/foo/admin/src/FooAdminAddCommand.php`**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = 'Description de la commande';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **Note**
> Pour éviter les conflits de commandes entre plugins, utiliser le format `vendor-plugin:commande`. Par ex., toutes les commandes de `foo/admin` doivent être préfixées par `foo-admin:`, ex. `foo-admin:add`.

### Ajouter la configuration
**Créer `config/plugin/foo/admin/command.php`**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // D'autres commandes au besoin...
];
```

> **Conseil**
> `command.php` enregistre les commandes personnalisées du plugin. Chaque entrée correspond à une classe de commande. `webman/console` les charge automatiquement. Voir [Commandes console](console.md).

### Exécuter l’export
Exécuter `php webman plugin:export --name=foo/admin` pour exporter et publier sur Packagist. Après installation de `foo/admin`, la commande `foo-admin:add` sera disponible. `php webman foo-admin:add jerry` affichera `Admin add jerry`.
