# Coda Stomp

Stomp è un protocollo di messaggistica testuale semplice (streaming) che fornisce un formato di connessione interoperabile, consentendo ai client STOMP di interagire con qualsiasi broker di messaggi STOMP. [workerman/stomp](https://github.com/walkor/stomp) implementa un client Stomp, utilizzato principalmente per scenari di code di messaggi come RabbitMQ, Apollo, ActiveMQ, ecc.

## Installazione
`composer require webman/stomp`

## Configurazione
Il file di configurazione si trova in `config/plugin/webman/stomp`

## Invio di messaggi
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // Coda
        $queue = 'examples';
        // Dati (se si passano array, è necessario serializzarli manualmente, ad es. con json_encode, serialize, ecc.)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // Eseguire l'invio
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> Per compatibilità con altri progetti, il componente Stomp non offre serializzazione e deserializzazione automatiche. Per dati in formato array è necessario serializzarli manualmente e deserializzarli in fase di consumo.

## Consumo di messaggi
Creare il file `app/queue/stomp/MyMailSend.php` (nome della classe libero, purché conforme allo standard PSR-4).
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // Nome della coda
    public $queue = 'examples';

    // Nome della connessione, corrispondente alla connessione in stomp.php
    public $connection = 'default';

    // Se il valore è 'client', occorre chiamare $ack_resolver->ack() per confermare al server il consumo avvenuto
    // Se il valore è 'auto', non è necessario chiamare $ack_resolver->ack()
    public $ack = 'auto';

    // Consumo
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // Se i dati sono un array, vanno deserializzati manualmente
        var_export(json_decode($data, true)); // stampa ['to' => 'tom@gmail.com', 'content' => 'hello']
        // Confermare al server il consumo avvenuto
        $ack_resolver->ack(); // questa chiamata può essere omessa quando ack è 'auto'
    }
}
```

# Abilitare il protocollo Stomp in RabbitMQ
Di default RabbitMQ non abilita il protocollo Stomp. Eseguire il seguente comando per abilitarlo:
```
rabbitmq-plugins enable rabbitmq_stomp
```
Dopo l'abilitazione, la porta predefinita per Stomp è 61613.
