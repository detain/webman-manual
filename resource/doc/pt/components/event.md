# Manipulação de eventos
`webman/event` oferece um mecanismo de eventos elegante que permite executar lógica de negócios sem alterar o código, alcançando o desacoplamento entre módulos. Cenário típico: quando um novo usuário se registra com sucesso, basta publicar um evento personalizado como `user.register`, e cada módulo pode receber o evento e executar a lógica correspondente.

## Instalação
`composer require webman/event`

## Inscrição em eventos
A inscrição em eventos é configurada de forma centralizada no arquivo `config/event.php`.

```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...outras funções de manipulação de eventos...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...outras funções de manipulação de eventos...
    ]
];
```

**Observação:**
- `user.register`, `user.logout`, etc. são nomes de eventos (tipo string). Recomenda-se palavras em minúsculas separadas por ponto (`.`).
- Um evento pode ter várias funções de manipulação; são chamadas na ordem configurada.

## Funções de manipulação de eventos
As funções podem ser métodos de classe, funções ou closures.

Exemplo: crie a classe `app/event/User.php` (crie o diretório se não existir).

```php
<?php
namespace app\event;
class User
{
    function register($user)
    {
        var_export($user);
    }
 
    function logout($user)
    {
        var_export($user);
    }
}
```

## Publicar eventos
Use `Event::dispatch($event_name, $data);` ou `Event::emit($event_name, $data);` para publicar um evento. Por exemplo:

```php
<?php
namespace app\controller;
use support\Request;
use Webman\Event\Event;
class User
{
    public function register(Request $request)
    {
        $user = [
            'name' => 'webman',
            'age' => 2
        ];
        Event::dispatch('user.register', $user);
    }
}
```

Existem duas funções para publicar: `Event::dispatch($event_name, $data);` e `Event::emit($event_name, $data);` — ambas com os mesmos parâmetros. A diferença: `emit` captura exceções internamente; se uma função lançar exceção, as outras ainda serão executadas. Já `dispatch` não captura exceções; se alguma função lançar exceção, a execução é interrompida e a exceção é propagada.

> **Dica**
> O parâmetro `$data` pode ser qualquer tipo de dado (array, instância de classe, string, etc.).

## Escuta de eventos com curinga
A inscrição com curinga permite tratar vários eventos com o mesmo listener. Por exemplo, em `config/event.php`:

```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```

É possível obter o nome específico do evento pelo segundo parâmetro `$event_data` da função de manipulação:

```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // nome específico do evento, ex. user.register, user.logout, etc.
        var_export($user);
    }
}
```

## Parar a transmissão do evento
Quando uma função de manipulação retorna `false`, a transmissão desse evento é interrompida.

## Manipulação de eventos com closures
A função de manipulação pode ser um método de classe ou uma closure. Por exemplo:

```php
<?php
return [
    'user.login' => [
        function($user){
            var_dump($user);
        }
    ]
];
```

## Ver eventos e ouvintes
Use o comando `php webman event:list` para ver todos os eventos e ouvintes configurados no projeto.

## Escopo de suporte
Além do projeto principal, os [plugins base](../plugin/base.md) e os [plugins de aplicação](../app/app.md) também suportam a configuração `event.php`.
**Arquivo de config. do plugin base:** `config/plugin/vendor/nome-plugin/event.php`
**Arquivo de config. do plugin de aplicação:** `plugin/nome-plugin/config/event.php`

## Notas
A manipulação de eventos não é assíncrona e não é adequada para operações lentas; estas devem usar filas de mensagens, como [webman/redis-queue](https://www.workerman.net/plugin/12).
