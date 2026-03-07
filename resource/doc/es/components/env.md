# Componente ENV vlucas/phpdotenv

## Descripción
`vlucas/phpdotenv` es un componente para cargar variables de entorno que permite distinguir configuraciones según el entorno (desarrollo, pruebas, etc.).

## Repositorio del proyecto

https://github.com/vlucas/phpdotenv
  
## Instalación
 
```php
composer require vlucas/phpdotenv
 ```
  
## Uso

### Crear un archivo `.env` en la raíz del proyecto
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### Modificar el archivo de configuración
**config/database.php**
```php
return [
    // Base de datos por defecto
    'default' => 'mysql',

    // Configuración de diversas bases de datos
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **Consejo**
> Se recomienda añadir el archivo `.env` al `.gitignore` para no subirlo al repositorio. Incluya un archivo de ejemplo `.env.example` en el repositorio. Al desplegar el proyecto, copie `.env.example` como `.env` y ajuste la configuración según el entorno. Así el proyecto cargará la configuración adecuada en cada entorno.

> **Nota**
> `vlucas/phpdotenv` puede tener errores con PHP en versión TS (Thread Safe). Use la versión NTS (Non-Thread-Safe). La versión actual de PHP se puede comprobar ejecutando `php -v`.

## Más información

Visite https://github.com/vlucas/phpdotenv
  
