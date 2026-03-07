# Processos personalizados

Em webman, você pode personalizar listeners ou processos da mesma forma que em workerman.

> **Nota**
> Usuários do Windows precisam usar `php windows.php` para iniciar o webman e poder executar processos personalizados.

## Serviço HTTP personalizado
Às vezes você pode ter uma necessidade especial de modificar o código principal do serviço HTTP do webman. Nesse caso, pode usar um processo personalizado.

Por exemplo, crie o arquivo `app\Server.php`.

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // Substitua aqui os métodos de Webman\App
}
```

Adicione a seguinte configuração em `config/process.php`.

```php
use Workerman\Worker;

return [
    // ... outras configurações omitidas ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Número de processos
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Classe de requisição
            'logger' => \support\Log::channel('default'), // Instância de log
            'appPath' => app_path(), // Localização do diretório app
            'publicPath' => public_path() // Localização do diretório public
        ]
    ]
];
```

> **Dica**
> Para desativar o processo HTTP incorporado do webman, basta definir `listen=>''` em `config/server.php`.

## Exemplo de listener WebSocket personalizado

Crie `app/Pusher.php`.

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

> Nota: todos os métodos onXXX devem ser públicos.

Adicione a seguinte configuração em `config/process.php`.

```php
return [
    // ... outras configurações de processo omitidas ...
    
    // websocket_test é o nome do processo
    'websocket_test' => [
        // Especifique aqui a classe do processo, ou seja, a classe Pusher definida acima
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## Exemplo de processo personalizado sem escuta

Crie `app/TaskTest.php`.

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // Verificar o banco de dados a cada 10 segundos para novos registros de usuários
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

Adicione a seguinte configuração em `config/process.php`.

```php
return [
    // ... outras configurações de processo omitidas ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> Nota: se listen for omitido, o processo não escutará em nenhuma porta; se count for omitido, o número de processos será 1 por padrão.

## Explicação do arquivo de configuração

Uma configuração completa de processo é definida da seguinte forma:

```php
return [
    // ... 
    
    // websocket_test é o nome do processo
    'websocket_test' => [
        // Especifique aqui a classe do processo
        'handler' => app\Pusher::class,
        // Protocolo, IP e porta para escutar (opcional)
        'listen'  => 'websocket://0.0.0.0:8888',
        // Número de processos (opcional, padrão 1)
        'count'   => 2,
        // Usuário para executar o processo (opcional, padrão usuário atual)
        'user'    => '',
        // Grupo de usuários para executar o processo (opcional, padrão grupo atual)
        'group'   => '',
        // Se o processo atual suporta reload (opcional, padrão true)
        'reloadable' => true,
        // Ativar reusePort
        'reusePort'  => true,
        // Transporte (opcional, defina como 'ssl' quando SSL for necessário, padrão 'tcp')
        'transport'  => 'tcp',
        // Contexto (opcional, passe o caminho do certificado quando o transporte for 'ssl')
        'context'    => [], 
        // Parâmetros do construtor da classe do processo (opcional)
        'constructor' => [],
        // Se este processo está habilitado
        'enable' => true
    ],
];
```

## Conclusão

Os processos personalizados do webman são, na prática, um encapsulamento simples do workerman. Eles separam a configuração da lógica de negócios e implementam os callbacks `onXXX` do workerman por meio de métodos de classe. Qualquer outro uso é idêntico ao workerman.
