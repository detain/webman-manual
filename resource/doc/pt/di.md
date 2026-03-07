# Injeção automática de dependências

No webman, a injeção automática de dependências é uma funcionalidade opcional e desativada por padrão. Se precisar de injeção automática de dependências, recomenda-se usar [php-di](https://php-di.org/doc/getting-started.html). Abaixo descreve-se o uso de `php-di` com webman.

## Instalação

```
composer require php-di/php-di:^7.0
```

Modifique a configuração `config/container.php`. O conteúdo final deve ser o seguinte:

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> O arquivo `config/container.php` deve retornar uma instância de contêiner em conformidade com a especificação PSR-11. Se não quiser usar `php-di`, pode criar e retornar aqui outra instância de contêiner compatível com PSR-11. A configuração padrão fornece apenas a funcionalidade básica do contêiner webman.

## Injeção por construtor

Crie o arquivo `app/service/Mailer.php` (crie o diretório se não existir) com o seguinte conteúdo:

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // Código de envio de e-mail omitido
    }
}
```

O conteúdo de `app/controller/UserController.php` é o seguinte:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{

    public function __construct(private Mailer $mailer)
    {
    }

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Olá e bem-vindo!');
        return response('ok');
    }
}
```

Normalmente, o seguinte código seria necessário para instanciar `app\controller\UserController`:

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

Usando `php-di`, os desenvolvedores não precisam instanciar manualmente o `Mailer` no controlador — o webman faz isso automaticamente. Se houver outras dependências de classes durante a instanciação do `Mailer`, o webman também as instanciará e injetará. Nenhum trabalho de inicialização é exigido do desenvolvedor.

> **Nota**
> Apenas instâncias criadas pelo framework ou `php-di` suportam injeção automática de dependências. Instâncias criadas manualmente com `new` não podem. Para injetar, use a interface `support\Container` em vez de `new`, por exemplo:

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// Instâncias criadas com new não podem usar injeção de dependências
$user_service = new UserService;
// Instâncias criadas com new não podem usar injeção de dependências
$log_service = new LogService($path, $name);

// Instâncias criadas com Container podem usar injeção de dependências
$user_service = Container::get(UserService::class);
// Instâncias criadas com Container podem usar injeção de dependências
$log_service = Container::make(LogService::class, [$path, $name]);
```

## Injeção por atributos

Além da injeção por construtor, pode usar injeção por atributos. Continuando o exemplo anterior, altere `app\controller\UserController` assim:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private Mailer $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Olá e bem-vindo!');
        return response('ok');
    }
}
```

Este exemplo usa o atributo `#[Inject]` para injeção e injeta automaticamente a instância na variável de membro com base no tipo de objeto. O efeito é igual à injeção por construtor, mas o código é mais conciso.

> **Nota**
> O webman não suporta injeção de parâmetros de controlador antes da versão 1.4.6. Por exemplo, o seguinte código não é suportado quando webman<=1.4.6:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // Injeção de parâmetros de controlador não suportada antes da versão 1.4.6
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'Olá e bem-vindo!');
        return response('ok');
    }
}
```

## Injeção personalizada por construtor

Por vezes os parâmetros passados ao construtor não são instâncias de classes mas strings, números, arrays ou outros dados não-objeto. Por exemplo, o construtor de Mailer pode precisar do IP e da porta do servidor SMTP:

```php
<?php
namespace app\service;

class Mailer
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // Código de envio de e-mail omitido
    }
}
```

Este caso não permite usar diretamente a injeção automática por construtor, pois `php-di` não consegue determinar os valores de `$smtp_host` e `$smtp_port`. Neste caso, pode usar injeção personalizada.

Adicione o seguinte código em `config/dependence.php` (crie o arquivo se não existir):

```php
return [
    // ... Outras configurações omitidas

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

Quando a injeção de dependências precisar obter uma instância de `app\service\Mailer`, usará automaticamente a instância criada nesta configuração.

Note que `config/dependence.php` usa `new` para instanciar a classe `Mailer`. Neste exemplo não há problema, mas se a classe `Mailer` depender de outras classes ou usar injeção por atributos internamente, a inicialização com `new` não fará a injeção automática de dependências. A solução é usar injeção personalizada por interface e inicializar as classes via `Container::get(nome da classe)` ou `Container::make(nome da classe, [parâmetros do construtor])`.

## Injeção personalizada por interface

Em projetos reais, prefere-se programar contra interfaces em vez de classes concretas. Por exemplo, `app\controller\UserController` deve depender de `app\service\MailerInterface` em vez de `app\service\Mailer`.

Defina a interface `MailerInterface`:

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

Defina a implementação de `MailerInterface`:

```php
<?php
namespace app\service;

class Mailer implements MailerInterface
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // Código de envio de e-mail omitido
    }
}
```

Use a interface `MailerInterface` em vez da implementação concreta:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\MailerInterface;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private MailerInterface $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Olá e bem-vindo!');
        return response('ok');
    }
}
```

Defina a implementação de `MailerInterface` em `config/dependence.php`:

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

Quando a aplicação precisar usar a interface `MailerInterface`, usará automaticamente a implementação `Mailer`.

> A vantagem de programar contra interfaces é que ao substituir um componente não é necessário alterar o código de negócios — apenas a implementação concreta em `config/dependence.php`. Isso também é muito útil para testes unitários.

## Outras injeções personalizadas

Além de dependências de classes, `config/dependence.php` pode definir outros valores como strings, números, arrays, etc.

Por exemplo, se `config/dependence.php` for definido assim:

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

Pode injetar `smtp_host` e `smtp_port` nas propriedades da classe com `#[Inject]`:

```php
<?php
namespace app\service;

use DI\Attribute\Inject;

class Mailer
{
    #[Inject("smtp_host")]
    private $smtpHost;

    #[Inject("smtp_port")]
    private $smtpPort;

    public function mail($email, $content)
    {
        // Código de envio de e-mail omitido
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // Exibirá 192.168.1.11:25
    }
}
```

# Carregamento preguiçoso (Lazy Loading)

> O carregamento preguiçoso é um padrão de projeto que adia a criação ou inicialização de objetos até serem realmente necessários.

Esta funcionalidade requer uma dependência adicional. O seguinte pacote é um fork de `ocramius/proxy-manager`; o repositório original não suporta PHP 8.

```
composer require friendsofphp/proxy-manager-lts
```

Uso:

```php
<?php

use DI\Attribute\Injectable;
use DI\Attribute\Inject;

#[Injectable(lazy: true)]
class MyClass
{
    private string $name;

    public function __construct()
    {
        echo "MyClass instanciado\n";
        $this->name = "Lazy Loaded Object";
    }

    public function getName(): string
    {
        return $this->name;
    }
}

class Controller
{
    #[Inject]
    public MyClass $myClass;

    public function getClass()
    {
        echo "Nome da classe proxy: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

Saída:

```
Nome da classe proxy: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass instanciado
name: Lazy Loaded Object
```

Este exemplo mostra que para uma classe declarada com o atributo `#[Injectable]`, ao ser injetada primeiro é criada a classe proxy. A classe real só é instanciada quando qualquer um dos seus métodos é chamado.

# Dependências circulares

Dependências circulares ocorrem quando várias classes dependem umas das outras, formando um ciclo fechado de dependências.

- Dependência circular direta
  - O módulo A depende do módulo B e o módulo B depende do módulo A
  - Forma o ciclo: A → B → A

- Dependência circular indireta
  - Envolve vários módulos em um ciclo de dependências
  - Por exemplo: A → B → C → A

Ao usar injeção por atributos, `php-di` detecta automaticamente dependências circulares e lança uma exceção. Se necessário, use a seguinte abordagem:

```php
class userController
{

    // Remova este código
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## Mais informações

Consulte a [documentação do php-di](https://php-di.org/doc/getting-started.html).
