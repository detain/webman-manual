# Initialisation des affaires

Parfois, nous avons besoin de faire une initialisation des affaires après le démarrage du processus, cette initialisation ne se fait qu'une seule fois pendant la durée de vie du processus, par exemple, après le démarrage du processus, nous pouvons définir une minuterie ou initialiser la connexion à la base de données, etc. Ci-dessous, nous expliquerons cela.

## Principe
Selon les explications dans **[process.md](./process.md)**, webman chargera les classes définies dans `config/bootstrap.php` (y compris `config/plugin/*/*/bootstrap.php`) après le démarrage du processus, puis exécutera la méthode `start` de ces classes. Nous pouvons ajouter du code métier dans la méthode `start` pour effectuer l'initialisation des affaires après le démarrage du processus.

## Processus
Supposons que nous voulions créer une minuterie pour signaler régulièrement l'utilisation de la mémoire par le processus. Cette classe est nommée `MemReport`.

#### Exécution de la commande

Exécutez la commande `php webman make:bootstrap MemReport` pour générer le fichier d'initialisation `app/bootstrap/MemReport.php`.

> **Conseil**
> Si votre webman n'a pas installé `webman/console`, exécutez la commande `composer require webman/console` pour l'installer.

#### Édition du fichier d'initialisation
Modifiez `app/bootstrap/MemReport.php`, le contenu ressemblera à ceci :
```php
<?php

namespace app\bootstrap;

use Webman\Bootstrap;

class MemReport implements Bootstrap
{
    public static function start($worker)
    {
        // Est-ce que c'est un environnement en ligne de commande ?
        $is_console = !$worker;
        if ($is_console) {
            // Si vous ne voulez pas exécuter cette initialisation dans l'environnement de ligne de commande, retournez simplement ici
            return;
        }
        
        // Exécute toutes les 10 secondes
        \Workerman\Timer::add(10, function () {
            // Pour des raisons de démonstration, nous utilisons une sortie pour simuler le processus de signalisation
            echo memory_get_usage() . "\n";
        });
        
    }

}
```

> **Conseil**
> Lors de l'utilisation de la ligne de commande, le framework exécutera également la méthode `start` définie dans `config/bootstrap.php`. Nous pouvons déterminer si c'est un environnement de ligne de commande en vérifiant si `$worker` est null, puis décider d'exécuter ou non le code d'initialisation des affaires.

#### Configuration au démarrage du processus
Ouvrez `config/bootstrap.php` et ajoutez la classe `MemReport` aux éléments de démarrage.
```php
return [
    // ... d'autres configurations omises ici ...
    
    app\bootstrap\MemReport::class,
];
```

De cette manière, nous avons terminé le processus d'initialisation des affaires.

## Remarques complémentaires
Les [processus personnalisés](../process.md) exécutent également la méthode `start` définie dans `config/bootstrap.php` après leur démarrage. Nous pouvons utiliser `$worker->name` pour déterminer quel processus est actuellement en cours d'exécution, et en outre `$worker->id` pour identifier le numéro du processus. Ainsi, nous pouvons décider d'exécuter ou non votre code d'initialisation des affaires dans ce processus. Par exemple, si nous n'avons besoin d'exécuter que dans le processus 0 de webman, le contenu de `MemReport.php` serait similaire à ce qui suit :
```php
<?php

namespace app\bootstrap;

use Webman\Bootstrap;

class MemReport implements Bootstrap
{
    public static function start($worker)
    {
        // Est-ce que c'est un environnement en ligne de commande ?
        $is_console = !$worker;
        if ($is_console) {
            // Si vous ne voulez pas exécuter cette initialisation dans l'environnement de ligne de commande, retournez simplement ici
            return;
        }

        // Exécuter uniquement dans le processus 0 de webman
        if ($worker->name != 'webman' || $worker->id != 0) {
            return;
        }
        
        // Exécute toutes les 10 secondes
        \Workerman\Timer::add(10, function () {
            // Pour des raisons de démonstration, nous utilisons une sortie pour simuler le processus de signalisation
            echo memory_get_usage() . "\n";
        });
        
    }

}
```
