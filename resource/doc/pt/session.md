# Gestão de sessão webman

## Exemplo
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

Obtenha uma instância de `Workerman\Protocols\Http\Session` através de `$request->session();` e use os métodos da instância para adicionar, modificar ou excluir dados da sessão.

> **Nota**
> Os dados da sessão são salvos automaticamente quando o objeto da sessão é destruído.
> Armazenar o objeto da sessão em uma variável global impede sua destruição e o salvamento automático. Nesse caso, chame manualmente `$session->save()` para salvar os dados.

## Obter todos os dados da sessão
```php
$session = $request->session();
$all = $session->all();
```
Retorna um array. Se não houver dados na sessão, retorna um array vazio.


## Obter um valor da sessão
```php
$session = $request->session();
$name = $session->get('name');
```
Retorna null se os dados não existirem.

Você pode passar um valor padrão como segundo argumento ao método `get`. Se o valor correspondente não for encontrado na sessão, o valor padrão será retornado. Por exemplo:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## Armazenar dados na sessão
Use o método `set` para armazenar um dado.
```php
$session = $request->session();
$session->set('name', 'tom');
```
O método `set` não retorna valor. Os dados são salvos automaticamente quando o objeto da sessão é destruído.

Use o método `put` para armazenar vários valores.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
Da mesma forma, o método `put` não retorna valor.

## Excluir dados da sessão
Use o método `forget` para excluir um ou mais dados da sessão.
```php
$session = $request->session();
// Excluir um item
$session->forget('name');
// Excluir vários itens
$session->forget(['name', 'age']);
```

O sistema também fornece o método `delete`. Ao contrário de `forget`, ele só pode excluir um item.
```php
$session = $request->session();
// Equivalente a $session->forget('name');
$session->delete('name');
```


## Obter e excluir um valor da sessão
```php
$session = $request->session();
$name = $session->pull('name');
```
Equivale ao seguinte código:
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
Retorna null se o valor da sessão correspondente não existir.


## Excluir todos os dados da sessão
```php
$request->session()->flush();
```
Não retorna valor. Os dados são removidos automaticamente do armazenamento quando o objeto da sessão é destruído.


## Verificar se um valor da sessão existe
```php
$session = $request->session();
$has = $session->has('name');
```
Retorna false se o valor não existir ou for null; caso contrário, retorna true.

```php
$session = $request->session();
$has = $session->exists('name');
```
O código acima também verifica a existência de um valor na sessão. A diferença é que `exists` retorna true mesmo quando o valor é null.

## Função auxiliar session()

webman fornece a função auxiliar `session()` para as mesmas operações.
```php
// Obter a instância da sessão
$session = session();
// Equivalente a
$session = $request->session();

// Obter um valor
$value = session('key', 'default');
// Equivalente a
$value = session()->get('key', 'default');
// Equivalente a
$value = $request->session()->get('key', 'default');

// Definir valores na sessão
session(['key1'=>'value1', 'key2' => 'value2']);
// Equivalente a
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// Equivalente a
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## Arquivo de configuração
O arquivo de configuração da sessão está em `config/session.php`. O conteúdo é semelhante a:
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class ou RedisSessionHandler::class ou RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // Quando handler é FileSessionHandler::class, o valor é 'file',
    // quando handler é RedisSessionHandler::class, o valor é 'redis'
    // quando handler é RedisClusterSessionHandler::class, o valor é 'redis_cluster' (cluster Redis)
    'type'    => 'file',

    // Diferentes handlers usam configurações diferentes
    'config' => [
        // Configuração quando type é 'file'
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // Configuração quando type é 'redis'
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // Nome do cookie para armazenar o session_id
    'auto_update_timestamp' => false,  // Atualizar automaticamente a sessão, padrão: desligado
    'lifetime' => 7*24*60*60,          // Tempo de expiração da sessão
    'cookie_lifetime' => 365*24*60*60, // Tempo de expiração do cookie que armazena o session_id
    'cookie_path' => '/',              // Caminho do cookie que armazena o session_id
    'domain' => '',                    // Domínio do cookie que armazena o session_id
    'http_only' => true,               // Habilitar httpOnly, padrão: habilitado
    'secure' => false,                 // Habilitar sessão apenas em HTTPS, padrão: desligado
    'same_site' => '',                 // Prevenir ataques CSRF e rastreamento de usuários, opções: strict/lax/none
    'gc_probability' => [1, 1000],     // Probabilidade de coleta de sessão
];
```

## Segurança
Não é recomendável armazenar diretamente instâncias de classes na sessão, especialmente de classes de fontes não confiáveis. A desserialização pode introduzir riscos de segurança.

