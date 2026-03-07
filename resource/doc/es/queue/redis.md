# Cola Redis

Cola de mensajes basada en Redis que admite el procesamiento diferido de mensajes.

## Instalación
`composer require webman/redis-queue`

## Archivo de configuración
El archivo de configuración de Redis se genera automáticamente en `{proyecto-principal}/config/plugin/webman/redis-queue/redis.php`, con un contenido similar al siguiente:
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // Contraseña, opcional
            'db' => 0,            // Base de datos
            'max_attempts'  => 5, // Número de reintentos tras fallo de consumo
            'retry_seconds' => 5, // Intervalo de reintento en segundos
        ]
    ],
];
```

### Reintento ante fallo de consumo
Si el consumo falla (ocurre una excepción), el mensaje se coloca en la cola diferida y espera al siguiente reintento. El número de reintentos se controla con `max_attempts`; el intervalo lo controlan conjuntamente `retry_seconds` y `max_attempts`. Por ejemplo, si `max_attempts` es 5 y `retry_seconds` es 10, el intervalo del 1.er reintento es `1*10` segundos, del 2.º `2*10` segundos, del 3.er `3*10` segundos, y así hasta 5 reintentos. Si se supera el número de reintentos configurado en `max_attempts`, el mensaje pasa a la cola de fallos con clave `{redis-queue}-failed`.

## Envío de mensajes (síncrono)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // Nombre de la cola
        $queue = 'send-mail';
        // Datos, se puede pasar un array directamente sin serializar
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Enviar mensaje
        Redis::send($queue, $data);
        // Enviar mensaje diferido, se procesará a los 60 segundos
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
En envío correcto, `Redis::send()` devuelve true; en caso contrario false o lanza una excepción.

> **Consejo**
> Puede haber desviación en el tiempo de consumo de la cola diferida. Por ejemplo, si la velocidad de consumo es menor que la de producción, la cola puede acumularse y retrasarse. Mitigación: ejecutar más procesos consumidores.

## Envío de mensajes (asíncrono)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // Nombre de la cola
        $queue = 'send-mail';
        // Datos, se puede pasar un array directamente sin serializar
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Enviar mensaje
        Client::send($queue, $data);
        // Enviar mensaje diferido, se procesará a los 60 segundos
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` no devuelve valor. Es un envío asíncrono y no garantiza la entrega al 100% a Redis.

> **Consejo**
> El principio de `Client::send()` es crear una cola en memoria local y sincronizar mensajes de forma asíncrona con Redis (la sincronización es rápida, unas 10.000 mensajes por segundo). Si el proceso se reinicia y los datos de la cola en memoria no se han sincronizado por completo, puede haber pérdida de mensajes. El envío asíncrono con `Client::send()` conviene para mensajes no críticos.

> **Consejo**
> `Client::send()` es asíncrono y solo puede usarse en el entorno de ejecución de Workerman. Para scripts de línea de comandos, use la interfaz síncrona `Redis::send()`.

## Enviar mensajes desde otros proyectos
A veces debe enviar mensajes desde otros proyectos y no puede usar `webman\redis-queue`. En esos casos puede usar la siguiente función para enviar mensajes a la cola.

```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting';
    $queue_delay = '{redis-queue}-delayed';
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```

Aquí el parámetro `$redis` es la instancia de Redis. Por ejemplo, el uso de la extensión redis es similar a:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## Consumo
El archivo de configuración del proceso consumidor está en `{proyecto-principal}/config/plugin/webman/redis-queue/process.php`. El directorio de consumidores está en `{proyecto-principal}/app/queue/redis/`.

El comando `php webman redis-queue:consumer my-send-mail` genera el archivo `{proyecto-principal}/app/queue/redis/MyMailSend.php`.

> **Consejo**
> Este comando requiere instalar el plugin [Consola](../plugin/console.md). Si prefiere no instalarlo, puede crear manualmente un archivo similar al siguiente:

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // Nombre de la cola a consumir
    public $queue = 'send-mail';

    // Nombre de conexión, corresponde a la conexión en plugin/webman/redis-queue/redis.php
    public $connection = 'default';

    // Consumo
    public function consume($data)
    {
        // No hace falta deserializar
        var_export($data); // Sale ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // Callback de fallo de consumo
    /* 
    $package = [
        'id' => 1357277951, // ID del mensaje
        'time' => 1709170510, // Hora del mensaje
        'delay' => 0, // Tiempo de retraso
        'attempts' => 2, // Número de consumos
        'queue' => 'send-mail', // Nombre de la cola
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // Contenido del mensaje
        'max_attempts' => 5, // Número máximo de reintentos
        'error' => 'Mensaje de error' // Mensaje de error
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // No hace falta deserializar
        var_export($package); 
    }
}
```

> **Nota**
> Se considera consumo correcto cuando no se lanza excepción ni Error durante el consumo; en caso contrario es fallo y el mensaje entra en la cola de reintentos. redis-queue no tiene mecanismo ack; se puede considerar ack automático (cuando no hay excepción ni Error). Para marcar el mensaje actual como no consumido con éxito, puede lanzar una excepción manualmente y enviar el mensaje a la cola de reintentos. En la práctica equivale a un mecanismo ack.

> **Consejo**
> Los consumidores admiten varios servidores y procesos, y el mismo mensaje **no** se consumirá dos veces. Los mensajes consumidos se eliminan automáticamente de la cola; no hace falta borrarlos manualmente.

> **Consejo**
> Los procesos consumidores pueden consumir varias colas distintas a la vez. Añadir una cola nueva no requiere cambiar la configuración en `process.php`. Para añadir un consumidor de cola nueva, basta con añadir la clase `Consumer` correspondiente bajo `app/queue/redis` y usar la propiedad `$queue` para indicar el nombre de la cola a consumir.

> **Consejo**
> Los usuarios de Windows deben ejecutar `php windows.php` para arrancar webman; si no, el proceso consumidor no se iniciará.

> **Consejo**
> El callback onConsumeFailure se dispara cada vez que falla el consumo. Aquí puede manejar la lógica tras el fallo. (Esta función requiere `webman/redis-queue>=1.3.2` y `workerman/redis-queue>=1.2.1`)

## Configurar procesos consumidores distintos para distintas colas
Por defecto todos los consumidores comparten el mismo proceso. A veces conviene separar el consumo de algunas colas—por ejemplo, negocios con consumo lento en un grupo de procesos y consumo rápido en otro. Para ello se pueden dividir los consumidores en dos directorios, p. ej. `app_path() . '/queue/redis/fast'` y `app_path() . '/queue/redis/slow'` (hay que actualizar el namespace de la clase consumidora). La configuración sería:
```php
return [
    ...otras configuraciones omitidas...
    
    'redis_consumer_fast'  => [ // La clave es personalizada, sin restricción de formato, aquí redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Directorio de clases consumidoras
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // La clave es personalizada, sin restricción de formato, aquí redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Directorio de clases consumidoras
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

Así los consumidores rápidos van al directorio `queue/redis/fast` y los lentos a `queue/redis/slow`, alcanzando el objetivo de asignar procesos consumidores a las colas.

## Configuración múltiple de Redis
#### Configuración
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // Contraseña, tipo string, opcional
            'db' => 0,            // Base de datos
            'max_attempts'  => 5, // Reintentos tras fallo de consumo
            'retry_seconds' => 5, // Intervalo de reintento en segundos
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // Contraseña, tipo string, opcional
            'db' => 0,            // Base de datos
            'max_attempts'  => 5, // Reintentos tras fallo de consumo
            'retry_seconds' => 5, // Intervalo de reintento en segundos
        ]
    ],
];
```

Nótese que se ha añadido una configuración Redis adicional con clave `other`.

#### Enviar mensajes a varios Redis

```php
// Enviar mensaje a la cola con clave `default`
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// Igual que
Client::send($queue, $data);
Redis::send($queue, $data);

// Enviar mensaje a la cola con clave `other`
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### Consumir de varios Redis
Consumir mensajes de la cola con clave `other` en la configuración:
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // Nombre de la cola a consumir
    public $queue = 'send-mail';

    // === Poner aquí 'other' para consumir de la cola con clave 'other' en la configuración ===
    public $connection = 'other';

    // Consumo
    public function consume($data)
    {
        // No hace falta deserializar
        var_export($data);
    }
}
```

## Preguntas frecuentes

**¿Por qué aparece el error `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)`?**

Este error solo ocurre con la interfaz de envío asíncrono `Client::send()`. El envío asíncrono guarda primero los mensajes en memoria local y luego los envía a Redis cuando el proceso está ocioso. Si Redis recibe mensajes más despacio de lo que se producen, o el proceso está ocupado con otras tareas y no tiene tiempo suficiente para sincronizar mensajes de memoria a Redis, puede acumularse un retraso. Si hay retraso de más de 600 segundos, se desencadena este error.

Solución: use la interfaz de envío síncrono `Redis::send()` para enviar mensajes.
