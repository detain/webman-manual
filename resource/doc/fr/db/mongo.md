# MongoDB

webman utilise par défaut [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) comme composant MongoDB. Ce composant a été extrait du projet Laravel et fonctionne de la même manière.

Vous devez d'abord installer l'extension MongoDB pour `php-cli` avant d'utiliser `mongodb/laravel-mongodb`.

> **Note**
> Utilisez la commande `php -m | grep mongodb` pour vérifier si l'extension MongoDB est installée pour `php-cli`. Notez que même si vous avez installé l'extension MongoDB pour `php-fpm`, cela ne signifie pas que vous pouvez l'utiliser pour `php-cli`, car `php-cli` et `php-fpm` sont des applications différentes et peuvent utiliser des fichiers de configuration `php.ini` différents. Utilisez la commande `php --ini` pour voir quel fichier de configuration `php.ini` est utilisé par votre `php-cli`.

## Installation

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

Après l'installation, un redémarrage est nécessaire (reload ne fonctionne pas).

## Configuration
Ajoutez une connexion `mongodb` dans le fichier `config/database.php`, similaire à ce qui suit :
```php
return [

    'default' => 'mysql',

    'connections' => [

         ...autres configurations omises ici...

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => '127.0.0.1',
            'port'     =>  27017,
            'database' => 'test',
            'username' => null,
            'password' => null,
            'options' => [
                // ici vous pouvez passer plus de paramètres au gestionnaire de pilote MongoDB
                // voir https://www.php.net/manual/en/mongodb-driver-manager.construct.php sous "Uri Options" pour une liste complète des paramètres que vous pouvez utiliser

                'appname' => 'homestead'
            ],
        ],
    ],
];
```

## Exemple
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        Db::connection('mongodb')->table('test')->insert([1,2,3]);
        return json(Db::connection('mongodb')->table('test')->get());
    }
}
```

## Exemple de modèle
```php
<?php
namespace app\model;

use DateTimeInterface;
use support\MongoModel as Model;

class Test extends Model
{
    protected $connection = 'mongodb';

    protected $table = 'test';

    public $timestamps = true;

    /**
     * @param DateTimeInterface $date
     * @return string
     */
    protected function serializeDate(DateTimeInterface $date): string
    {
        return $date->format('Y-m-d H:i:s');
    }
}
```

## Pour plus d'informations, veuillez visiter

https://github.com/mongodb/laravel-mongodb
