# Cache

[webman/cache](https://github.com/webman-php/cache) é um componente de cache baseado em [symfony/cache](https://github.com/symfony/cache), compatível com ambientes de corotina e não corotina, com suporte a pool de conexões.

## Instalação

```php
composer require -W webman/cache
```

## Exemplo
```php
<?php
namespace app\controller;

use support\Request;
use support\Cache;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Cache::set($key, rand());
        return response(Cache::get($key));
    }
}
```

## Localização do arquivo de configuração
O arquivo de configuração está em `config/cache.php`. Crie-o manualmente se não existir.

## Conteúdo do arquivo de configuração
```php
<?php
return [
    'default' => 'file',
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default'
        ],
        'array' => [
            'driver' => 'array'
        ],
        'apcu' => [
            'driver' => 'apcu'
        ]
    ]
];
```
`stores.driver` suporta quatro drivers: **file**, **redis**, **array** e **apcu**.

#### Driver file
Driver padrão. Sem dependências externas. Suporta compartilhamento de cache entre processos. Não suporta compartilhamento entre múltiplos servidores.

#### Driver array
Armazenamento em memória com melhor desempenho, mas consome memória. Não suporta compartilhamento entre processos ou servidores. Os dados são perdidos ao reiniciar o processo. Típico para projetos com volume de cache pequeno.

#### Driver apcu
Armazenamento em memória. Desempenho inferior apenas ao array. Suporta compartilhamento de cache entre processos. Não suporta compartilhamento entre múltiplos servidores. Os dados são perdidos ao reiniciar o processo. Típico para projetos com volume de cache pequeno.

> Requer a instalação e ativação da [extensão APCu](https://pecl.php.net/package/APCu). Não recomendado para cenários com escrita/exclusão frequente de cache, pois pode causar degradação significativa de desempenho.

#### Driver redis
Depende do componente [webman/redis](./redis.md). Suporta compartilhamento de cache entre processos e servidores.

**stores.redis.connection**

`stores.redis.connection` corresponde à chave definida em `config/redis.php`. Ao usar Redis, reutiliza a configuração de `webman/redis`, incluindo o pool de conexões.

**Recomenda-se adicionar uma configuração Redis dedicada ao cache em `config/redis.php`, por exemplo:**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== Adicionar novo
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

Em seguida, defina `stores.redis.connection` como `cache`. O arquivo `config/cache.php` final deve ficar assim:

```php
<?php
return [
    'default' => 'redis', // <====
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache' // <====
        ],
        'array' => [
            'driver' => 'array'
        ]
    ]
];
```

## Trocar de armazenamento
É possível alternar manualmente o armazenamento para usar drivers diferentes, por exemplo:

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **Dica**
> Os nomes das chaves de cache são restringidos por [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) e não devem conter nenhum destes caracteres: `{}()/\@:`. A partir de `symfony/cache` 7.2.4, essa verificação pode ser ignorada configurando a opção PHP ini `zend.assertions=-1`.

## Usar outros componentes de cache

Consulte [outros bancos de dados](others.md#ThinkCache) para o componente [ThinkCache](https://github.com/webman-php/think-cache).
