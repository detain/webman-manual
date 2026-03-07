# Archivos de configuración

## Ubicación
Los archivos de configuración de webman se encuentran en el directorio `config/`. En el proyecto puede obtener la configuración correspondiente mediante la función `config()`.

## Obtener configuración

Obtener toda la configuración:
```php
config();
```

Obtener toda la configuración en `config/app.php`:
```php
config('app');
```

Obtener la configuración `debug` en `config/app.php`:
```php
config('app.debug');
```

Si la configuración es un array, puede obtener valores anidados usando `.`. Por ejemplo:
```php
config('file.key1.key2');
```

## Valor por defecto
```php
config($key, $default);
```
Pase el valor por defecto como segundo parámetro. Si la configuración no existe, se devolverá el valor por defecto. Si la configuración no existe y no se ha establecido valor por defecto, se devuelve `null`.

## Configuración personalizada
Los desarrolladores pueden agregar sus propios archivos de configuración en el directorio `config/`. Por ejemplo:

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**Uso al obtener la configuración**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## Modificar la configuración
webman no admite la modificación dinámica de la configuración. Toda la configuración debe modificarse manualmente en el archivo correspondiente y luego recargar (reload) o reiniciar (restart).

> **Nota**
> La configuración del servidor `config/server.php` y la de procesos `config/process.php` no admiten reload. Debe reiniciar para que los cambios surtan efecto.

## Recordatorio importante
Si crea archivos de configuración en un subdirectorio de `config/`, por ejemplo `config/order/status.php`, necesita un archivo `app.php` en el directorio `config/order/` con el siguiente contenido:
```php
<?php
return [
    'enable' => true,
];
```
`enable` en `true` indica al marco que cargue la configuración de este directorio.
La estructura del directorio de configuración debe ser así:
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
Así puede acceder al array o claves específicas de `status.php` mediante `config.order.status`.


## Referencia de archivos de configuración

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // Puerto de escucha (eliminado desde 1.6.0, configurar en config/process.php)
    'transport' => 'tcp', // Protocolo de transporte (eliminado desde 1.6.0, configurar en config/process.php)
    'context' => [], // SSL etc. (eliminado desde 1.6.0, configurar en config/process.php)
    'name' => 'webman', // Nombre del proceso (eliminado desde 1.6.0, configurar en config/process.php)
    'count' => cpu_count() * 4, // Número de procesos (eliminado desde 1.6.0, configurar en config/process.php)
    'user' => '', // Usuario (eliminado desde 1.6.0, configurar en config/process.php)
    'group' => '', // Grupo (eliminado desde 1.6.0, configurar en config/process.php)
    'reusePort' => false, // Reutilización de puerto (eliminado desde 1.6.0, configurar en config/process.php)
    'event_loop' => '',  // Clase del bucle de eventos, selección automática por defecto
    'stop_timeout' => 2, // Tiempo máximo de espera al recibir señal stop/restart/reload, salida forzada si excede
    'pid_file' => runtime_path() . '/webman.pid', // Ubicación del archivo PID
    'status_file' => runtime_path() . '/webman.status', // Ubicación del archivo de estado
    'stdout_file' => runtime_path() . '/logs/stdout.log', // Ubicación stdout, toda la salida tras iniciar webman
    'log_file' => runtime_path() . '/logs/workerman.log', // Ubicación del log de Workerman
    'max_package_size' => 10 * 1024 * 1024 // Tamaño máximo de paquete, 10M. Limita el tamaño de subida
];
```

#### app.php
```php
return [
    'debug' => true,  // Modo debug, habilita stack trace en errores. Desactivar en producción
    'error_reporting' => E_ALL, // Nivel de reporte de errores
    'default_timezone' => 'Asia/Shanghai', // Zona horaria por defecto
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // Ruta del directorio public
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // Ruta del directorio runtime
    'controller_suffix' => 'Controller', // Sufijo del controlador
    'controller_reuse' => false, // Reutilización de controladores
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // Configuración de procesos webman
    'webman' => [ 
        'handler' => Http::class, // Clase manejadora del proceso
        'listen' => 'http://0.0.0.0:8787', // Dirección de escucha
        'count' => cpu_count() * 4, // Número de procesos, 4x CPU por defecto
        'user' => '', // Usuario del proceso, usar usuario con pocos privilegios
        'group' => '', // Grupo del proceso, usar grupo con pocos privilegios
        'reusePort' => false, // Habilitar reusePort, distribuye conexiones entre workers
        'eventLoop' => '', // Clase del bucle de eventos, usa server.event_loop si vacío
        'context' => [], // Contexto de escucha, p.ej. SSL
        'constructor' => [ // Parámetros del constructor del manejador, aquí clase Http
            'requestClass' => Request::class, // Clase de petición personalizada
            'logger' => Log::channel('default'), // Instancia del logger
            'appPath' => app_path(), // Ruta del directorio app
            'publicPath' => public_path() // Ruta del directorio public
        ]
    ],
    // Proceso monitor para detección de cambios de archivos y fugas de memoria
    'monitor' => [
        'handler' => app\process\Monitor::class, // Clase manejadora
        'reloadable' => false, // Este proceso no ejecuta reload
        'constructor' => [ // Parámetros del constructor del manejador
            // Directorios a vigilar, evitar demasiados (ralentiza la detección)
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Extensiones de archivo a vigilar
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // Otras opciones
            'options' => [
                // Habilitar monitoreo de archivos, solo Linux, desactivado por defecto en modo daemon
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // Habilitar monitoreo de memoria, solo Linux
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// Devolver una instancia del contenedor de inyección de dependencias PSR-11
return new Webman\Container;
```

#### dependence.php
```php
// Configurar servicios y dependencias en el contenedor de inyección de dependencias
return [];
```

#### route.php
```php

use support\Route;
// Definir ruta para la ruta /test
Route::any('/test', function (Request $request) {
    return response('test');
});
```

#### view.php
```php
use support\view\Raw;
use support\view\Twig;
use support\view\Blade;
use support\view\ThinkPHP;

return [
    'handler' => Raw::class // Clase manejadora de vistas por defecto
];
```

### autoload.php
```php
// Configurar archivos de autocarga del marco
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php',
        base_path() . '/support/Response.php',
    ]
];
```

#### cache.php
```php
// Configuración de caché
return [
    'default' => 'file', // Driver por defecto
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // Ruta de almacenamiento de caché
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // Nombre de conexión Redis, referencia redis.php
        ],
        'array' => [
            'driver' => 'array' // Caché en memoria, se borra al reiniciar
        ]
    ]
];
```

#### redis.php
```php
return [
    'default' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 0,
    ],
];
```

#### database.php
```php
return [
 // Base de datos por defecto
 'default' => 'mysql',
 // Configuraciones de conexión
 'connections' => [

     'mysql' => [
         'driver'      => 'mysql',
         'host'        => '127.0.0.1',
         'port'        => 3306,
         'database'    => 'webman',
         'username'    => 'webman',
         'password'    => '',
         'unix_socket' => '',
         'charset'     => 'utf8',
         'collation'   => 'utf8_unicode_ci',
         'prefix'      => '',
         'strict'      => true,
         'engine'      => null,
     ],

     'sqlite' => [
         'driver'   => 'sqlite',
         'database' => '',
         'prefix'   => '',
     ],

     'pgsql' => [
         'driver'   => 'pgsql',
         'host'     => '127.0.0.1',
         'port'     => 5432,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
         'schema'   => 'public',
         'sslmode'  => 'prefer',
     ],

     'sqlsrv' => [
         'driver'   => 'sqlsrv',
         'host'     => 'localhost',
         'port'     => 1433,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
     ],
 ],
];
```

#### exception.php
```php
return [
    // Establecer clase manejadora de excepciones
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // Manejador
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // Nombre del archivo de log
                    7, //$maxFiles // Mantener logs de 7 días
                    Monolog\Logger::DEBUG, // Nivel de log
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // Formateador
                    'constructor' => [null, 'Y-m-d H:i:s', true], // Parámetros del formateador
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // Tipo
    'type' => 'file', // or redis or redis_cluster
     // Manejador
    'handler' => FileSessionHandler::class,
     // Configuración
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // Directorio de almacenamiento
        ],
        'redis' => [
            'host' => '127.0.0.1',
            'port' => 6379,
            'auth' => '',
            'timeout' => 2,
            'database' => '',
            'prefix' => 'redis_session_',
        ],
        'redis_cluster' => [
            'host' => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth' => '',
            'prefix' => 'redis_session_',
        ]
    ],
    'session_name' => 'PHPSID', // Nombre de sesión
    'auto_update_timestamp' => false, // Actualizar timestamp automáticamente, evitar expiración
    'lifetime' => 7*24*60*60, // Vida útil
    'cookie_lifetime' => 365*24*60*60, // Vida útil de cookie
    'cookie_path' => '/', // Ruta de cookie
    'domain' => '', // Dominio de cookie
    'http_only' => true, // Solo HTTP
    'secure' => false, // Solo HTTPS
    'same_site' => '', // Atributo SameSite
    'gc_probability' => [1, 1000], // Probabilidad de recolección de sesiones
];
```

#### middleware.php
```php
// Configurar middleware
return [];
```

#### static.php
```php
return [
    'enable' => true, // Habilitar servicio de archivos estáticos de webman
    'middleware' => [ // Middleware de archivos estáticos, para política de caché, CORS etc.
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // Idioma por defecto
    'locale' => 'zh_CN',
    // Idiomas de respaldo
    'fallback_locale' => ['zh_CN', 'en'],
    // Ubicación de archivos de traducción
    'path' => base_path() . '/resource/translations',
];
```
