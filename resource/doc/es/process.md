# Procesos personalizados

En webman puedes personalizar listeners o procesos igual que en workerman.

> **Nota**
> Los usuarios de Windows deben usar `php windows.php` para iniciar webman y poder ejecutar procesos personalizados.

## Servicio HTTP personalizado
A veces puedes tener necesidades especiales que requieren modificar el código principal del servicio HTTP de webman. En ese caso puedes usar un proceso personalizado.

Por ejemplo, crea el archivo `app\Server.php`.

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // Aquí se sobrescriben los métodos de Webman\App
}
```

Añade la siguiente configuración en `config/process.php`.

```php
use Workerman\Worker;

return [
    // ... otras configuraciones omitidas ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Número de procesos
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Clase de petición
            'logger' => \support\Log::channel('default'), // Instancia de log
            'appPath' => app_path(), // Ubicación del directorio app
            'publicPath' => public_path() // Ubicación del directorio public
        ]
    ]
];
```

> **Consejo**
> Para desactivar el proceso HTTP incorporado de webman, basta con definir `listen=>''` en `config/server.php`.

## Ejemplo de listener WebSocket personalizado

Crea `app/Pusher.php`.

```php
<?php
namespace app;

use Workerman\Connection\TcpConnection;

class Pusher
{
    public function onConnect(TcpConnection $connection)
    {
        echo "onConnect\n";
    }

    public function onWebSocketConnect(TcpConnection $connection, $http_buffer)
    {
        echo "onWebSocketConnect\n";
    }

    public function onMessage(TcpConnection $connection, $data)
    {
        $connection->send($data);
    }

    public function onClose(TcpConnection $connection)
    {
        echo "onClose\n";
    }
}
```

> Nota: Todos los métodos onXXX deben ser públicos.

Añade la siguiente configuración en `config/process.php`.

```php
return [
    // ... otras configuraciones de proceso omitidas ...
    
    // websocket_test es el nombre del proceso
    'websocket_test' => [
        // Aquí se especifica la clase del proceso, la clase Pusher definida arriba
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## Ejemplo de proceso sin escucha

Crea `app/TaskTest.php`.

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // Comprobar cada 10 segundos si hay nuevos usuarios registrados
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

Añade la siguiente configuración en `config/process.php`.

```php
return [
    // ... otras configuraciones de proceso omitidas ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> Nota: Si se omite listen, el proceso no escuchará en ningún puerto. Si se omite count, el número de procesos por defecto es 1.

## Explicación del archivo de configuración

La configuración completa de un proceso se define así:

```php
return [
    // ... 

    // websocket_test es el nombre del proceso
    'websocket_test' => [
        // Aquí se especifica la clase del proceso
        'handler' => app\Pusher::class,
        // Protocolo, IP y puerto de escucha (opcional)
        'listen'  => 'websocket://0.0.0.0:8888',
        // Número de procesos (opcional, por defecto 1)
        'count'   => 2,
        // Usuario de ejecución del proceso (opcional, por defecto el actual)
        'user'    => '',
        // Grupo de ejecución del proceso (opcional, por defecto el actual)
        'group'   => '',
        // Si el proceso admite recarga (opcional, por defecto true)
        'reloadable' => true,
        // Activar reusePort
        'reusePort'  => true,
        // Transporte (opcional, usar 'ssl' si se necesita SSL, por defecto 'tcp')
        'transport'  => 'tcp',
        // Contexto (opcional, indicar ruta del certificado si transport es 'ssl')
        'context'    => [], 
        // Parámetros del constructor de la clase de proceso (opcional)
        'constructor' => [],
        // Si este proceso está habilitado
        'enable' => true
    ],
];
```

## Conclusión

Los procesos personalizados en webman son esencialmente una encapsulación sencilla de workerman. Separan la configuración de la lógica de negocio e implementan los callbacks `onXXX` de workerman mediante métodos de clase. El resto del uso es idéntico a workerman.
