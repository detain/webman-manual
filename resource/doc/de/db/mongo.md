# MongoDB

webman verwendet standardmäßig [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) als MongoDB-Komponente. Diese wurde aus dem Laravel-Projekt extrahiert und funktioniert wie Laravel.

Vor der Verwendung von `mongodb/laravel-mongodb` müssen Sie die MongoDB-Erweiterung für `php-cli` installieren.

> **Hinweis**
> Verwenden Sie den Befehl `php -m | grep mongodb`, um zu überprüfen, ob die MongoDB-Erweiterung für `php-cli` installiert ist. Beachten Sie: Selbst wenn Sie die MongoDB-Erweiterung für `php-fpm` installiert haben, bedeutet dies nicht, dass Sie sie in `php-cli` verwenden können, da `php-cli` und `php-fpm` unterschiedliche Anwendungen sind und möglicherweise unterschiedliche `php.ini`-Konfigurationen verwenden. Verwenden Sie den Befehl `php --ini`, um festzustellen, welche `php.ini`-Konfigurationsdatei von Ihrem `php-cli` verwendet wird.

## Installation

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

Nach der Installation ist ein Neustart erforderlich (reload funktioniert nicht).

## Konfiguration
Fügen Sie in der Datei `config/database.php` die `mongodb`-Verbindung hinzu, ähnlich wie folgt:
```php
return [

    'default' => 'mysql',

    'connections' => [

         ...andere Konfigurationen hier ausgelassen...

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => '127.0.0.1',
            'port'     =>  27017,
            'database' => 'test',
            'username' => null,
            'password' => null,
            'options' => [
                // Hier können Sie weitere Einstellungen an den Mongo Driver Manager übergeben
                // siehe https://www.php.net/manual/en/mongodb-driver-manager.construct.php unter "Uri Options" für eine Liste der vollständigen Parameter, die Sie verwenden können

                'appname' => 'homestead'
            ],
        ],
    ],
];
```

## Beispiel
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

## Modellbeispiel
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

## Weitere Informationen finden Sie unter

https://github.com/mongodb/laravel-mongodb
