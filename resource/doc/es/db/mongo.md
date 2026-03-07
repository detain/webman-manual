# MongoDB

webman utiliza [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) como componente MongoDB de forma predeterminada. Está extraído del proyecto Laravel y se utiliza de la misma manera.

Antes de utilizar `mongodb/laravel-mongodb`, es necesario instalar la extensión MongoDB para `php-cli`.

> **Nota**
> Utiliza el comando `php -m | grep mongodb` para verificar si la extensión MongoDB está instalada para `php-cli`. Ten en cuenta que incluso si has instalado la extensión MongoDB para `php-fpm`, no significa que esté disponible para `php-cli`, ya que `php-cli` y `php-fpm` son aplicaciones diferentes que pueden utilizar configuraciones distintas de `php.ini`. Utiliza el comando `php --ini` para verificar qué archivo de configuración `php.ini` está utilizando tu `php-cli`.

## Instalación

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

Después de la instalación, es necesario reiniciar (reload no es suficiente).

## Configuración
Agrega la conexión `mongodb` en el archivo `config/database.php`, de la siguiente manera:
```php
return [

    'default' => 'mysql',

    'connections' => [

         ...otras configuraciones omitidas aquí...

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => '127.0.0.1',
            'port'     =>  27017,
            'database' => 'test',
            'username' => null,
            'password' => null,
            'options' => [
                // aquí puedes pasar más configuraciones al administrador del controlador MongoDB
                // consulta https://www.php.net/manual/en/mongodb-driver-manager.construct.php en "Uri Options" para ver una lista de parámetros completos que puedes utilizar

                'appname' => 'homestead'
            ],
        ],
    ],
];
```

## Ejemplo
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

## Ejemplo de modelo
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

## Para más información, visita

https://github.com/mongodb/laravel-mongodb
