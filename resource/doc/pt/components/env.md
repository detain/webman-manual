# Componente ENV vlucas/phpdotenv

## Descrição
`vlucas/phpdotenv` é um componente de carregamento de variáveis de ambiente, usado para diferenciar configurações em ambientes diferentes (desenvolvimento, testes, etc.).

## Repositório do projeto

https://github.com/vlucas/phpdotenv
  
## Instalação
 
```php
composer require vlucas/phpdotenv
 ```
  
## Uso

### Criar um arquivo `.env` na raiz do projeto
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### Modificar o arquivo de configuração
**config/database.php**
```php
return [
    // Banco de dados padrão
    'default' => 'mysql',

    // Configurações de diversas bases de dados
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **Dica**
> Recomenda-se adicionar o arquivo `.env` ao `.gitignore` para não commitá-lo no repositório. Adicione um arquivo de exemplo `.env.example` no repositório. Ao implantar o projeto, copie `.env.example` como `.env` e ajuste a configuração conforme o ambiente. Assim o projeto carregará configurações diferentes em cada ambiente.

> **Atenção**
> `vlucas/phpdotenv` pode ter bugs com PHP em versão TS (Thread Safe). Use a versão NTS (Non-Thread-Safe). A versão atual do PHP pode ser verificada executando `php -v`.

## Mais informações

Visite https://github.com/vlucas/phpdotenv
  
