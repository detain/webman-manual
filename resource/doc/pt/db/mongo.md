# MongoDB

webman utiliza por padrão o [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) como componente MongoDB. Foi extraído do projeto Laravel e funciona da mesma forma.

Antes de usar `mongodb/laravel-mongodb`, é necessário instalar a extensão MongoDB para `php-cli`.

> **Nota**
> Use o comando `php -m | grep mongodb` para verificar se a extensão MongoDB está instalada para `php-cli`. Atenção: mesmo que você tenha instalado a extensão MongoDB para `php-fpm`, isso não significa que você possa usá-la para `php-cli`, pois `php-cli` e `php-fpm` são aplicativos diferentes e podem usar arquivos de configuração `php.ini` diferentes. Use o comando `php --ini` para verificar qual arquivo de configuração `php.ini` é usado pelo seu `php-cli`.

## Instalação

```php
composer require -W webman/database mongodb/laravel-mongodb ^4.8
```

Após a instalação, é necessário reiniciar (reload não é suficiente).

## Configuração
No arquivo `config/database.php`, adicione uma conexão `mongodb`, semelhante ao seguinte:
```php
return [

    'default' => 'mysql',

    'connections' => [

         ...configurações omitidas aqui...

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => '127.0.0.1',
            'port'     =>  27017,
            'database' => 'test',
            'username' => null,
            'password' => null,
            'options' => [
                // aqui você pode passar mais configurações para o Mongo Driver Manager
                // consulte https://www.php.net/manual/en/mongodb-driver-manager.construct.php em "Uri Options" para uma lista de parâmetros completos que você pode usar

                'appname' => 'homestead'
            ],
        ],
    ],
];
```

## Exemplo
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

## Exemplo de modelo
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

## Para mais informações, visite

https://github.com/mongodb/laravel-mongodb
