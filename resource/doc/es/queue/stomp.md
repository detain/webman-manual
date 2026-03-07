# Cola STOMP

STOMP es un protocolo simple de mensajería orientado a texto (streaming) que proporciona un formato de conexión interoperable, permitiendo a los clientes STOMP interactuar con cualquier intermediario de mensajes (broker) STOMP. [workerman/stomp](https://github.com/walkor/stomp) implementa el cliente Stomp y se utiliza principalmente en escenarios de colas de mensajes como RabbitMQ, Apollo, ActiveMQ, etc.

## Instalación
`composer require webman/stomp`

## Configuración
El archivo de configuración se encuentra en `config/plugin/webman/stomp`

## Envío de mensajes
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // Cola
        $queue = 'examples';
        // Datos (al enviar un array, hay que serializarlo manualmente, por ejemplo con json_encode, serialize, etc.)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // Ejecutar el envío
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> Por compatibilidad con otros proyectos, el componente Stomp no ofrece serialización ni deserialización automática. Si se envían datos en forma de array, hay que serializarlos manualmente y deserializarlos al consumir.

## Consumo de mensajes
Crea el archivo `app/queue/stomp/MyMailSend.php` (el nombre de la clase puede ser cualquiera, siempre que cumpla el estándar PSR-4).
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // Nombre de la cola
    public $queue = 'examples';

    // Nombre de la conexión, correspondiente a la conexión en stomp.php
    public $connection = 'default';

    // Si el valor es 'client', hay que llamar a $ack_resolver->ack() para notificar al servidor el consumo correcto
    // Si el valor es 'auto', no hace falta llamar a $ack_resolver->ack()
    public $ack = 'auto';

    // Consumir
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // Si los datos son un array, hay que deserializarlos manualmente
        var_export(json_decode($data, true)); // imprime ['to' => 'tom@gmail.com', 'content' => 'hello']
        // Notificar al servidor el consumo correcto
        $ack_resolver->ack(); // cuando ack es 'auto', esta llamada puede omitirse
    }
}
```

# Habilitar el protocolo STOMP en RabbitMQ
Por defecto, RabbitMQ no tiene habilitado el protocolo STOMP. Hay que ejecutar el siguiente comando para habilitarlo:
```
rabbitmq-plugins enable rabbitmq_stomp
```
Una vez habilitado, el puerto predeterminado de STOMP es 61613.
