# Base de datos Medoo

[webman/medoo](https://github.com/webman-php/medoo) extiende [Medoo](https://medoo.in/) con pool de conexiones y funciona tanto en entornos con corutinas como sin ellas. El uso es el mismo que Medoo.

## Instalación
`composer require webman/medoo`

## Configuración de la base de datos Medoo
Ubicación del archivo de configuración: `config/plugin/webman/medoo/database.php`

## Uso de la base de datos Medoo
```php
<?php
namespace app\controller;

use support\Request;
use support\Medoo;

class Index
{
    public function index(Request $request)
    {
        $user = Medoo::get('user', '*', ['uid' => 1]);
        return json($user);
    }
}
```

> **Nota**
> `Medoo::get('user', '*', ['uid' => 1]);`
> equivale a
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## Configuración de múltiples bases de datos Medoo

**Configuración**
Añada una nueva configuración en `config/plugin/webman/medoo/database.php` con cualquier clave; aquí utilizamos `other`.

```php
<?php
return [
    'default' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [ // Configuración del pool de conexiones
            'max_connections' => 5, // Número máximo de conexiones
            'min_connections' => 1, // Número mínimo de conexiones
            'wait_timeout' => 60,   // Tiempo máximo de espera al obtener una conexión; excepción si se supera
            'idle_timeout' => 3,    // Tiempo máximo de inactividad de conexiones en el pool; las que lo superen se cierran hasta llegar a min_connections
            'heartbeat_interval' => 50, // Intervalo de latido del pool en segundos; se recomienda menos de 60 segundos
        ]
    ],
    // Añadir aquí la configuración 'other'
    'other' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [
            'max_connections' => 5,
            'min_connections' => 1,
            'wait_timeout' => 60,
            'idle_timeout' => 3,
            'heartbeat_interval' => 50,
        ],
    ],
];
```

## Uso de la base de datos Medoo
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

Consulte la [documentación oficial de Medoo](https://medoo.in/api/select)
