# Banco de dados Medoo

[webman/medoo](https://github.com/webman-php/medoo) estende [Medoo](https://medoo.in/) com pool de conexões e funciona em ambientes coroutine e não-coroutine. O uso é o mesmo do Medoo.

## Instalação
`composer require webman/medoo`

## Configuração do banco de dados Medoo
Local do arquivo de configuração: `config/plugin/webman/medoo/database.php`

## Uso do banco de dados Medoo
```php
<?php
namespace app\controller;

use support\Request;
use support\Medoo;

class Index
{
    public function index(Request $request)
    {
        $user = Medoo::get('user', '*', ['uid' => 1]);
        return json($user);
    }
}
```

> **Dica**
> `Medoo::get('user', '*', ['uid' => 1]);`
> equivale a
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## Configuração de múltiplos bancos de dados Medoo

**Configuração**
Adicione uma nova configuração em `config/plugin/webman/medoo/database.php` com qualquer chave; aqui usamos `other`.

```php
<?php
return [
    'default' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [ // Configuração do pool de conexões
            'max_connections' => 5, // Número máximo de conexões
            'min_connections' => 1, // Número mínimo de conexões
            'wait_timeout' => 60,   // Tempo máximo de espera ao obter uma conexão do pool; exceção se excedido
            'idle_timeout' => 3,    // Tempo máximo de inatividade das conexões no pool; as que excederem são fechadas até min_connections
            'heartbeat_interval' => 50, // Intervalo de heartbeat do pool em segundos; recomendado menos de 60 segundos
        ]
    ],
    // Adicionar configuração 'other' aqui
    'other' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [
            'max_connections' => 5,
            'min_connections' => 1,
            'wait_timeout' => 60,
            'idle_timeout' => 3,
            'heartbeat_interval' => 50,
        ],
    ],
];
```

## Uso do banco de dados Medoo
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

Consulte a [documentação oficial do Medoo](https://medoo.in/api/select)
