# Processamento de operações lentas

Às vezes precisamos processar operações lentas para evitar que afetem o processamento de outras solicitações no webman. Essas operações podem utilizar diferentes soluções de processamento conforme a situação.

## Opção 1: Uso de fila de mensagens
Consulte [fila Redis](../queue/redis.md) [fila STOMP](../queue/stomp.md)

#### Vantagens
Pode lidar com picos repentinos de solicitações de processamento

#### Desvantagens
Não pode retornar resultados diretamente ao cliente. Se for necessário enviar resultados, é preciso coordenar com outros serviços, como usar [webman/push](https://www.workerman.net/plugin/2) para enviar os resultados do processamento.

## Opção 2: Adicionar uma nova porta HTTP

Adicionar uma nova porta HTTP para processar solicitações lentas. Essas solicitações lentas acessam esta porta e são processadas por um grupo específico de processos, retornando os resultados diretamente ao cliente após o processamento.

#### Vantagens
Pode retornar os dados diretamente ao cliente

#### Desvantagens
Não pode lidar com picos repentinos de solicitações

#### Passos de implementação
Adicione a seguinte configuração em `config/process.php`.
```php
return [
    // ... Outras configurações omitidas aqui ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Número de processos
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Configuração da classe de solicitação
            'logger' => \support\Log::channel('default'), // Instância do logger
            'appPath' => app_path(), // Localização do diretório app
            'publicPath' => public_path() // Localização do diretório public
        ]
    ]
];
```

Dessa forma, as interfaces lentas podem ser processadas pelo grupo de processos em `http://127.0.0.1:8686/` sem afetar o processamento de outras operações nos demais processos.

Para que o frontend não perceba a diferença de portas, é possível adicionar um proxy para a porta 8686 no nginx. Supondo que os caminhos das solicitações lentas comecem com `/task`, a configuração do nginx seria semelhante à seguinte:
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# Adicionar um novo upstream 8686
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # Solicitações que começam com /task vão para a porta 8686, altere /task para o prefixo desejado conforme necessário
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # Outras solicitações vão para a porta 8787 original
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

Dessa forma, quando o cliente acessar `dominio.com/task/xxx`, será processado pela porta 8686 separada sem afetar o processamento de solicitações na porta 8787.

## Opção 3: Uso de HTTP Chunked para envio assíncrono de dados em segmentos

#### Vantagens
Pode retornar os dados diretamente ao cliente

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
            $connection->send(new Chunk('')); // Enviar chunk vazio para indicar fim da resposta
        });
        // Enviar primeiro os cabeçalhos HTTP, os dados são enviados de forma assíncrona
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **Nota**
> Este exemplo usa o cliente `workerman/http-client` para obter resultados HTTP de forma assíncrona e retornar os dados. Também é possível usar outros clientes assíncronos, como [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html).
