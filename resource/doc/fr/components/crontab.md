# Composant de tâches planifiées Crontab

## Description

`workerman/crontab` est similaire au crontab Linux, à la différence qu'il prend en charge la planification à la seconde près.

Format du temps :

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[optionnel ; si absent, la granularité minimale est la minute]
```

## URL du projet

https://github.com/walkor/crontab
  
## Installation
 
```php
composer require workerman/crontab
```
  
## Utilisation

**Étape 1 : Créer le fichier de processus `app/process/Task.php`**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {
    
        // Exécuter chaque seconde
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Exécuter toutes les 5 secondes
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Exécuter chaque minute
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Exécuter toutes les 5 minutes
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Exécuter à la première seconde de chaque minute
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // Exécuter à 7h50 chaque jour (la position des secondes est omise ici)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**Étape 2 : Configurer le processus pour démarrer avec webman**
  
Ouvrir le fichier de configuration `config/process.php` et ajouter ce qui suit :

```php
return [
    ....autre configuration omise....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**Étape 3 : Redémarrer webman**

> Note : Les tâches planifiées ne s'exécutent pas immédiatement ; elles commencent à la minute suivante.

## Remarques
Crontab n'est pas asynchrone. Exemple : un processus task configure deux timers A et B, tous deux exécutés chaque seconde. Si la tâche A prend 10 secondes, B doit attendre la fin de A avant de s'exécuter, ce qui retarde B.
Si la logique est sensible aux intervalles temporels, exécuter les tâches sensibles au temps dans des processus séparés pour éviter les interférences. Exemple pour `config/process.php` :

```php
return [
    ....autre configuration omise....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
Placer les tâches sensibles au temps dans `process/Task1.php` et les autres dans `process/Task2.php`.

Pour en savoir plus sur `config/process.php`, voir [Processus personnalisés](../process.md).
