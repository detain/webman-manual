# Procesamiento de operaciones lentas

A veces necesitamos procesar operaciones lentas para evitar que afecten el procesamiento de otras solicitudes en webman. Según la situación, estas operaciones pueden utilizar diferentes soluciones de procesamiento.

## Opción 1: Uso de colas de mensajes
Consulte [cola Redis](../queue/redis.md) [cola STOMP](../queue/stomp.md)

#### Ventajas
Puede manejar picos repentinos de solicitudes de procesamiento

#### Desventajas
No puede devolver resultados directamente al cliente. Si necesita enviar resultados, debe coordinarse con otros servicios, como usar [webman/push](https://www.workerman.net/plugin/2) para enviar los resultados del procesamiento.

## Opción 2: Añadir un nuevo puerto HTTP

Añadir un nuevo puerto HTTP para manejar solicitudes lentas. Estas solicitudes lentas se procesan accediendo a este puerto por un grupo específico de procesos, y los resultados se devuelven directamente al cliente tras el procesamiento.

#### Ventajas
Puede devolver los datos directamente al cliente

#### Desventajas
No puede manejar picos repentinos de solicitudes

#### Pasos de implementación
Añada la siguiente configuración en `config/process.php`.
```php
return [
    // ... Otras configuraciones omitidas aquí ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Número de procesos
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Configuración de la clase de solicitud
            'logger' => \support\Log::channel('default'), // Instancia del logger
            'appPath' => app_path(), // Ubicación del directorio app
            'publicPath' => public_path() // Ubicación del directorio public
        ]
    ]
];
```

De esta forma, las interfaces lentas pueden pasar por el grupo de procesos en `http://127.0.0.1:8686/` sin afectar el procesamiento de otras operaciones en los demás procesos.

Para que el frontend no perciba la diferencia de puertos, puede añadir un proxy al puerto 8686 en nginx. Suponiendo que las rutas de las solicitudes lentas comienzan con `/task`, la configuración de nginx sería similar a la siguiente:
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# Añadir un nuevo upstream 8686
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # Las solicitudes que comienzan con /task van al puerto 8686, cambie /task al prefijo que necesite según su caso
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # Las demás solicitudes van al puerto 8787 original
  location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      if (!-f $request_filename){
          proxy_pass http://webman;
      }
  }
}
```

De esta forma, cuando el cliente acceda a `dominio.com/task/xxx`, será procesado por el puerto 8686 independiente sin afectar el procesamiento de solicitudes en el puerto 8787.

## Opción 3: Uso de HTTP Chunked para envío asíncrono de datos segmentados

#### Ventajas
Puede devolver los datos directamente al cliente

**Instalar workerman/http-client**

```
composer require workerman/http-client
```

**app/controller/IndexController.php**
```php
<?php
namespace app\controller;

use support\Request;
use support\Response;
use Workerman\Protocols\Http\Chunk;

class IndexController
{
    public function index(Request $request)
    {
        $connection = $request->connection;
        $http = new \Workerman\Http\Client();
        $http->get('https://example.com/', function ($response) use ($connection) {
            $connection->send(new Chunk($response->getBody()));
            $connection->send(new Chunk('')); // Enviar chunk vacío para indicar fin de la respuesta
        });
        // Enviar primero los encabezados HTTP, los datos se envían de forma asíncrona
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **Nota**
> Este ejemplo utiliza el cliente `workerman/http-client` para obtener resultados HTTP de forma asíncrona y devolver los datos. También puede utilizar otros clientes asíncronos como [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html).
