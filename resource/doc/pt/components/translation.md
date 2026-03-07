# Multilíngue

A funcionalidade multilíngue utiliza o componente [symfony/translation](https://github.com/symfony/translation).

## Instalação
```
composer require symfony/translation
```

## Criar pacotes de idioma
O webman armazena os pacotes de idioma por padrão no diretório `resource/translations` (crie-o se não existir). Para alterar o diretório, configure em `config/translation.php`.
Cada idioma corresponde a uma subpasta, e as definições ficam em `messages.php` por padrão. Exemplo:
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

Todos os arquivos de idioma retornam um array, por exemplo:
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## Configuração

`config/translation.php`

```php
return [
    // Idioma padrão
    'locale' => 'zh_CN',
    // Idioma alternativo: quando a tradução não for encontrada no idioma atual, tenta o idioma alternativo
    'fallback_locale' => ['zh_CN', 'en'],
    // Pasta onde os arquivos de idioma são armazenados
    'path' => base_path() . '/resource/translations',
];
```

## Tradução

Use o método `trans()` para traduzir.

Criar o arquivo de idioma `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 世界!',
];
```

Criar o arquivo `app/controller/UserController.php`:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        $hello = trans('hello'); // 你好 世界!
        return response($hello);
    }
}
```

Ao acessar `http://127.0.0.1:8787/user/get` será retornado "你好 世界!"

## Alterar o idioma padrão

Use o método `locale()` para trocar de idioma.

Adicionar o arquivo de idioma `resource/translations/en/messages.php`:
```php
return [
    'hello' => 'hello world!',
];
```

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // Trocar idioma
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
Ao acessar `http://127.0.0.1:8787/user/get` será retornado "hello world!"

Também é possível usar o 4º parâmetro da função `trans()` para trocar temporariamente de idioma. O exemplo acima é equivalente a:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // O 4º parâmetro troca o idioma
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## Definir o idioma explicitamente para cada requisição
translation é um singleton, ou seja, todas as requisições compartilham a mesma instância. Se uma requisição definir o idioma padrão com `locale()`, isso afetará todas as requisições seguintes do processo. Por isso, é necessário definir o idioma explicitamente para cada requisição, por exemplo com o seguinte middleware:

Criar o arquivo `app/middleware/Lang.php` (criar o diretório se não existir):
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Lang implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        locale(session('lang', 'zh_CN'));
        return $handler($request);
    }
}
```

Adicionar o middleware global em `config/middleware.php`:
```php
return [
    // Middleware global
    '' => [
        // ... outros middlewares omitidos
        app\middleware\Lang::class,
    ]
];
```


## Usar marcadores de posição
Às vezes uma mensagem contém variáveis que precisam ser traduzidas, por exemplo
```php
trans('hello ' . $name);
```
Nesses casos, use marcadores de posição.

Alterar `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 %name%!',
];
```
Ao traduzir, passe os valores pelo segundo parâmetro:
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## Tratar plurais
Em alguns idiomas a estrutura da frase varia conforme a quantidade. Por exemplo, "There is %count% apple" está correto quando `%count%` é 1, mas incorreto quando é maior.

Nesses casos, use o **pipe** (`|`) para listar as formas no plural.

Adicionar `apple_count` no arquivo `resource/translations/en/messages.php`:
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

Também é possível especificar intervalos numéricos para regras de plural mais complexas:
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## Especificar arquivo de idioma

O nome padrão é `messages.php`, mas é possível criar arquivos com outros nomes.

Criar o arquivo `resource/translations/zh_CN/admin.php`:
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

Especificar o arquivo pelo 3º parâmetro de `trans()` (omitir a extensão `.php`):
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## Mais informações
Consulte a [documentação symfony/translation](https://symfony.com/doc/current/translation.html)
