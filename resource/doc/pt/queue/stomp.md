# Fila Stomp

Stomp é um protocolo simples de mensagens orientado a texto (streaming) que fornece um formato de conexão interoperável, permitindo que clientes STOMP interajam com qualquer corretor de mensagens STOMP (Broker). [workerman/stomp](https://github.com/walkor/stomp) implementa um cliente Stomp, utilizado principalmente em cenários de filas de mensagens como RabbitMQ, Apollo, ActiveMQ, etc.

## Instalação
`composer require webman/stomp`

## Configuração
O arquivo de configuração está em `config/plugin/webman/stomp`

## Envio de mensagens
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // Fila
        $queue = 'examples';
        // Dados (ao enviar um array, é necessário serializar manualmente, por ex. com json_encode, serialize, etc.)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // Executar o envio
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> Por compatibilidade com outros projetos, o componente Stomp não fornece serialização nem deserialização automáticas. Se os dados enviados forem um array, é necessário serializar manualmente e deserializar ao consumir.

## Consumo de mensagens
Crie o arquivo `app/queue/stomp/MyMailSend.php` (o nome da classe pode ser arbitrário, desde que cumpra o padrão PSR-4).
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // Nome da fila
    public $queue = 'examples';

    // Nome da conexão, correspondente à conexão em stomp.php
    public $connection = 'default';

    // Quando o valor for 'client', é necessário chamar $ack_resolver->ack() para informar o servidor que consumiu com sucesso
    // Quando o valor for 'auto', não é necessário chamar $ack_resolver->ack()
    public $ack = 'auto';

    // Consumir
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // Se os dados forem um array, é necessário deserializar manualmente
        var_export(json_decode($data, true)); // output ['to' => 'tom@gmail.com', 'content' => 'hello']
        // Informar o servidor que consumiu com sucesso
        $ack_resolver->ack(); // quando ack for 'auto', esta chamada pode ser omitida
    }
}
```

# Ativar o protocolo Stomp no RabbitMQ
O RabbitMQ não ativa o protocolo Stomp por defeito. Execute o seguinte comando para ativar:
```
rabbitmq-plugins enable rabbitmq_stomp
```
Após a ativação, a porta padrão do Stomp é 61613.
