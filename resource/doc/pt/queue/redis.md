# Fila Redis

Fila de mensagens baseada em Redis que suporta processamento de mensagens com atraso.

## Instalação
`composer require webman/redis-queue`

## Arquivo de configuração
O arquivo de configuração Redis é gerado automaticamente em `{projeto-principal}/config/plugin/webman/redis-queue/redis.php`, com conteúdo similar ao seguinte:
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // Senha, opcional
            'db' => 0,            // Banco de dados
            'max_attempts'  => 5, // Número de tentativas após falha no consumo
            'retry_seconds' => 5, // Intervalo de tentativa em segundos
        ]
    ],
];
```

### Nova tentativa após falha no consumo
Se o consumo falhar (ocorrer exceção), a mensagem será colocada na fila de atraso e aguardará a próxima tentativa. O número de tentativas é controlado por `max_attempts`; o intervalo, por `retry_seconds` e `max_attempts`. Ex.: se `max_attempts` for 5 e `retry_seconds` for 10, o intervalo da 1.ª tentativa é `1*10` s, da 2.ª `2*10` s, da 3.ª `3*10` s, até 5 tentativas. Se o número de tentativas exceder o definido em `max_attempts`, a mensagem vai para a fila de falhas com chave `{redis-queue}-failed`.

## Envio de mensagem (síncrono)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // Nome da fila
        $queue = 'send-mail';
        // Dados, podem ser passados como array diretamente, sem serialização
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Enviar mensagem
        Redis::send($queue, $data);
        // Enviar mensagem atrasada, processada após 60 segundos
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
Em sucesso, `Redis::send()` retorna true; caso contrário false ou exceção.

> **Dica**
> Pode haver desvio no horário de consumo da fila atrasada. Ex.: quando a velocidade de consumo é menor que a de produção, a fila pode acumular e atrasar. Mitigação: rodar mais processos consumidores.

## Envio de mensagem (assíncrono)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // Nome da fila
        $queue = 'send-mail';
        // Dados, podem ser passados como array diretamente, sem serialização
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Enviar mensagem
        Client::send($queue, $data);
        // Enviar mensagem atrasada, processada após 60 segundos
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` não retorna valor. É um envio assíncrono e não garante 100% de entrega ao Redis.

> **Dica**
> O princípio de `Client::send()` é criar uma fila em memória local e sincronizar mensagens de forma assíncrona com Redis (sincronização é rápida, ~10.000 mensagens/s). Se o processo reiniciar antes dos dados da fila em memória estarem totalmente sincronizados, mensagens podem ser perdidas. O envio assíncrono com `Client::send()` é adequado para mensagens não críticas.

> **Dica**
> `Client::send()` é assíncrono e só pode ser usado no ambiente de execução Workerman. Para scripts em linha de comando use a interface síncrona `Redis::send()`.

## Enviar mensagens de outros projetos
Às vezes é preciso enviar mensagens de outros projetos e não se pode usar `webman\redis-queue`. Nesses casos, pode usar a função abaixo para enviar mensagens à fila.

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

Aqui o parâmetro `$redis` é a instância Redis. Ex.: o uso da extensão redis é similar a:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## Consumo
O arquivo de configuração do processo consumidor está em `{projeto-principal}/config/plugin/webman/redis-queue/process.php`. O diretório dos consumidores está em `{projeto-principal}/app/queue/redis/`.

O comando `php webman redis-queue:consumer my-send-mail` gera o arquivo `{projeto-principal}/app/queue/redis/MyMailSend.php`.

> **Dica**
> Este comando requer a instalação do plugin [Console](../plugin/console.md). Se não quiser instalar, pode criar manualmente um arquivo similar ao seguinte:

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // Nome da fila a consumir
    public $queue = 'send-mail';

    // Nome da conexão, corresponde à conexão em plugin/webman/redis-queue/redis.php
    public $connection = 'default';

    // Consumo
    public function consume($data)
    {
        // Não precisa deserializar
        var_export($data); // Saída ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // Callback de falha no consumo
    /* 
    $package = [
        'id' => 1357277951, // ID da mensagem
        'time' => 1709170510, // Tempo da mensagem
        'delay' => 0, // Tempo de atraso
        'attempts' => 2, // Número de consumos
        'queue' => 'send-mail', // Nome da fila
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // Conteúdo da mensagem
        'max_attempts' => 5, // Número máximo de tentativas
        'error' => 'Mensagem de erro' // Mensagem de erro
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // Não precisa deserializar
        var_export($package); 
    }
}
```

> **Nota**
> O consumo é considerado bem-sucedido quando nenhuma exceção ou Error é lançada durante o consumo; caso contrário é falha e a mensagem entra na fila de nova tentativa. O redis-queue não tem mecanismo ack; pode ser visto como ack automático (quando não há exceção ou Error). Para marcar a mensagem atual como não consumida com sucesso, lance uma exceção manualmente para enviar à fila de nova tentativa. Na prática equivale a um mecanismo ack.

> **Dica**
> Os consumidores suportam múltiplos servidores e processos, e a mesma mensagem **não** será consumida duas vezes. As mensagens consumidas são removidas automaticamente da fila; não é necessário remover manualmente.

> **Dica**
> Os processos consumidores podem consumir várias filas diferentes ao mesmo tempo. Adicionar uma nova fila não exige alterar a configuração em `process.php`. Para adicionar consumidor de fila nova, basta adicionar a classe `Consumer` correspondente em `app/queue/redis` e usar a propriedade `$queue` para especificar o nome da fila a consumir.

> **Dica**
> Usuários Windows precisam executar `php windows.php` para iniciar o webman; caso contrário o processo consumidor não será iniciado.

> **Dica**
> O callback onConsumeFailure é acionado a cada falha no consumo. Aqui você pode tratar a lógica pós-falha. (Esta função requer `webman/redis-queue>=1.3.2` e `workerman/redis-queue>=1.2.1`)

## Definir processos consumidores diferentes para filas diferentes
Por padrão todos os consumidores compartilham o mesmo processo. Às vezes quer-se separar o consumo de algumas filas—ex.: negócios de consumo lento em um grupo de processos, consumo rápido em outro. Para isso, os consumidores podem ser divididos em dois diretórios, ex.: `app_path() . '/queue/redis/fast'` e `app_path() . '/queue/redis/slow'` (o namespace da classe consumidora deve ser atualizado). A configuração fica:
```php
return [
    ...outras configurações omitidas...
    
    'redis_consumer_fast'  => [ // A chave é personalizada, sem restrição de formato, aqui redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Diretório das classes consumidoras
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // A chave é personalizada, sem restrição de formato, aqui redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Diretório das classes consumidoras
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

Assim, consumidores de negócios rápidos ficam em `queue/redis/fast` e os lentos em `queue/redis/slow`, atingindo o objetivo de atribuir processos consumidores às filas.

## Configuração múltipla de Redis
#### Configuração
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // Senha, tipo string, opcional
            'db' => 0,            // Banco de dados
            'max_attempts'  => 5, // Tentativas após falha no consumo
            'retry_seconds' => 5, // Intervalo de tentativa em segundos
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // Senha, tipo string, opcional
            'db' => 0,            // Banco de dados
            'max_attempts'  => 5, // Tentativas após falha no consumo
            'retry_seconds' => 5, // Intervalo de tentativa em segundos
        ]
    ],
];
```

A configuração inclui uma configuração Redis adicional com chave `other`.

#### Enviar mensagens para Redis múltiplos

```php
// Enviar mensagem para a fila com chave `default`
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// O mesmo que
Client::send($queue, $data);
Redis::send($queue, $data);

// Enviar mensagem para a fila com chave `other`
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### Consumir de Redis múltiplos
Consumir mensagens da fila com chave `other` na configuração:
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // Nome da fila a consumir
    public $queue = 'send-mail';

    // === Definir 'other' aqui para consumir da fila com chave 'other' na configuração ===
    public $connection = 'other';

    // Consumo
    public function consume($data)
    {
        // Não precisa deserializar
        var_export($data);
    }
}
```

## FAQ

**Por que ocorre o erro `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)`?**

Este erro ocorre apenas com a interface de envio assíncrono `Client::send()`. O envio assíncrono salva primeiro as mensagens na memória local e depois as envia ao Redis quando o processo está ocioso. Se o Redis recebe mensagens mais devagar do que são produzidas, ou o processo está ocupado com outras tarefas e não tem tempo para sincronizar as mensagens da memória ao Redis, pode haver acúmulo. Se as mensagens ficarem acumuladas por mais de 600 segundos, este erro é acionado.

Solução: use a interface de envio síncrono `Redis::send()` para envio de mensagens.
