# Guia rápido do banco de dados (baseado no componente Laravel)

[webman/database](https://github.com/webman-php/database) é baseado em [illuminate/database](https://github.com/illuminate/database) e adiciona pool de conexões para ambientes com e sem corrotinas. O uso é igual ao do Laravel.

Você também pode consultar [Uso de outros componentes de banco de dados](others.md) para usar ThinkPHP ou outros bancos.

## Instalação do banco de dados

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

É necessário reiniciar após a instalação (reload não funciona).

> **Dica**
> webman/database depende do `illuminate/database` do Laravel, então as dependências são instaladas automaticamente.

> **Nota**
> Se não precisar de paginação, eventos de banco de dados ou registro de SQL, basta executar:
> `composer require -W webman/database`

## Configuração do banco de dados
`config/database.php`
```php

return [
    // Banco de dados padrão
    'default' => 'mysql',

    // Configurações de conexões
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'test',
            'username'    => 'root',
            'password'    => '',
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
            'options' => [
                PDO::ATTR_EMULATE_PREPARES => false, // Necessário ao usar swoole ou swow
            ],
            'pool' => [ // Configuração do pool de conexões
                'max_connections' => 5, // Número máximo de conexões
                'min_connections' => 1, // Número mínimo de conexões
                'wait_timeout' => 3,    // Tempo máximo de espera ao obter conexão do pool; excede → exceção. Só em ambiente corrotina
                'idle_timeout' => 60,   // Tempo máximo de ociosidade das conexões; depois são fechadas até min_connections
                'heartbeat_interval' => 50, // Intervalo de heartbeat do pool em segundos; recomendado < 60
            ],
        ],
    ],
];
```

Exceto a configuração `pool`, o restante é igual ao Laravel.

## Sobre o pool de conexões
* Cada processo tem seu próprio pool; os pools não são compartilhados entre processos.
* Sem corrotinas as requisições são executadas em sequência, não há concorrência, então o pool terá no máximo uma conexão.
* Com corrotinas as requisições rodam em paralelo; o pool ajusta o número de conexões dinamicamente, sem exceder `max_connections` nem ficar abaixo de `min_connections`.
* Como o pool está limitado a `max_connections`, quando há mais corrotinas usando o banco, algumas aguardam na fila até `wait_timeout` segundos; excedendo, uma exceção é lançada.
* Em ociosidade (com ou sem corrotinas), as conexões são devolvidas após `idle_timeout` até atingir `min_connections` (`min_connections` pode ser 0).


## Exemplo de uso do banco de dados
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        $default_uid = 29;
        $uid = $request->get('uid', $default_uid);
        $name = Db::table('users')->where('uid', $uid)->value('username');
        return response("hello $name");
    }
}
```

O uso é igual ao Laravel: o método `Db::table()` para operar no banco de dados.
