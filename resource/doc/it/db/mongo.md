# MongoDB

webman utilizza per impostazione predefinita [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) come componente MongoDB. È stato estratto dal progetto Laravel e funziona allo stesso modo.

Prima di utilizzare `mongodb/laravel-mongodb`, è necessario installare l'estensione MongoDB per `php-cli`.

> **Nota**
> Utilizza il comando `php -m | grep mongodb` per verificare se l'estensione MongoDB è installata per `php-cli`. Nota: anche se hai installato l'estensione MongoDB per `php-fpm`, non significa che sia disponibile per `php-cli`, poiché sono due applicazioni diverse con configurazioni `php.ini` diverse. Utilizza il comando `php --ini` per verificare quale file di configurazione `php.ini` è utilizzato da `php-cli`.

## Installazione

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

Dopo l'installazione, è necessario riavviare (reload non è sufficiente).

## Configurazione
Aggiungi la connessione `mongodb` nel file `config/database.php`, simile a quanto segue:
```php
return [

    'default' => 'mysql',

    'connections' => [

         ...altre configurazioni omesse qui...

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => '127.0.0.1',
            'port'     =>  27017,
            'database' => 'test',
            'username' => null,
            'password' => null,
            'options' => [
                // qui puoi passare altre impostazioni al Mongo Driver Manager
                // consulta https://www.php.net/manual/en/mongodb-driver-manager.construct.php nel paragrafo "Uri Options" per un elenco completo dei parametri che puoi utilizzare

                'appname' => 'homestead'
            ],
        ],
    ],
];
```

## Esempio
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

## Esempio di modello
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

## Per ulteriori informazioni, visitare

https://github.com/mongodb/laravel-mongodb
