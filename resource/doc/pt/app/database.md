# Banco de dados

Como a maioria dos plugins instala o [webman-admin](https://www.workerman.net/plugin/82), recomenda-se reutilizar diretamente a configuração do banco de dados do `webman-admin`.

Os modelos cuja classe base é `plugin\admin\app\model\Base` utilizarão automaticamente a configuração do banco de dados do webman-admin.
```php
<?php

namespace plugin\foo\app\model;

use plugin\admin\app\model\Base;

class Orders extends Base
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'foo_orders';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $primaryKey = 'id';
    
}
```

Também é possível acessar o banco de dados do webman-admin via `plugin.admin.mysql`, por exemplo:

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## Usar seu próprio banco de dados

Os plugins também podem optar por usar seu próprio banco de dados. Por exemplo, o conteúdo de `plugin/foo/config/database.php`:

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql é o nome da conexão
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'banco_de_dados',
            'username'    => 'nome_de_usuario',
            'password'    => 'senha',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin é o nome da conexão
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'banco_de_dados',
            'username'    => 'nome_de_usuario',
            'password'    => 'senha',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

O formato de referência é `Db::connection('plugin.{plugin}.{nome_conexao}');`, por exemplo:

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

Para usar o banco de dados do projeto principal, basta chamá-lo diretamente:

```php
use support\Db;
Db::table('user')->first();
// Supondo que o projeto principal também tenha uma conexão admin configurada
Db::connection('admin')->table('admin')->first();
```

#### Configurar o banco de dados para o Model

Você pode criar uma classe Base para o Model e definir a propriedade `$connection` para usar a conexão com o banco de dados do próprio plugin:

```php
<?php

namespace plugin\foo\app\model;

use DateTimeInterface;
use support\Model;

class Base extends Model
{
    /**
     * @var string
     */
    protected $connection = 'plugin.foo.mysql';

}
```

Assim, todos os modelos do plugin que herdam de Base usarão automaticamente o banco de dados do próprio plugin.

## Importação automática do banco de dados

A execução de `php webman app-plugin:create foo` cria o plugin foo, incluindo `plugin/foo/api/Install.php` e `plugin/foo/install.sql`.

> **Dica**
> Se o arquivo install.sql não for gerado, tente atualizar o `webman/console` com: `composer require webman/console ^1.3.6`

#### Importar o banco de dados ao instalar o plugin

Ao instalar um plugin, o método `install` em Install.php é executado e lança automaticamente as instruções SQL em `install.sql`, importando assim as tabelas do banco de dados.

O conteúdo de `install.sql` deve incluir a criação de tabelas e alterações históricas do esquema. Cada instrução deve terminar com `;`, por exemplo:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Chave primária',
  `order_id` varchar(50) NOT NULL COMMENT 'ID do pedido',
  `user_id` int NOT NULL COMMENT 'ID do usuário',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Valor a pagar',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Pedidos';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Chave primária',
  `name` varchar(50) NOT NULL COMMENT 'Nome',
  `price` int NOT NULL COMMENT 'Preço',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Produtos';
```

**Alterar a conexão do banco de dados**

Por padrão, `install.sql` é importado no banco de dados do webman-admin. Para importar em outro banco de dados, altere a propriedade `$connection` em `Install.php`:

```php
<?php

class Install
{
    // Especificar o banco de dados do próprio plugin
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**Teste**

Execute `php webman app-plugin:install foo` para instalar o plugin. Depois, verifique o banco de dados: as tabelas `foo_orders` e `foo_goods` devem ter sido criadas.

#### Alterar a estrutura das tabelas durante a atualização do plugin

Quando uma atualização do plugin exige novas tabelas ou alterações no esquema, adicione as instruções SQL correspondentes ao final de `install.sql`. Cada instrução deve terminar com `;`. Por exemplo, adicionar a tabela `foo_user` e a coluna `status` em `foo_orders`:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Chave primária',
  `order_id` varchar(50) NOT NULL COMMENT 'ID do pedido',
  `user_id` int NOT NULL COMMENT 'ID do usuário',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Valor a pagar',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Pedidos';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Chave primária',
 `name` varchar(50) NOT NULL COMMENT 'Nome',
 `price` int NOT NULL COMMENT 'Preço',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Produtos';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Chave primária',
 `name` varchar(50) NOT NULL COMMENT 'Nome'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Usuário';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT 'Status';
```

Durante a atualização, o método `update` em Install.php executa as instruções de `install.sql`. As novas são executadas; as já aplicadas são ignoradas, aplicando assim corretamente as alterações no banco de dados durante as atualizações.

#### Excluir o banco de dados ao desinstalar o plugin

Ao desinstalar um plugin, o método `uninstall` em Install.php é chamado. Ele analisa automaticamente as instruções CREATE TABLE em `install.sql` e exclui essas tabelas.

Se quiser executar apenas seu próprio `uninstall.sql` em vez da exclusão automática de tabelas, crie `plugin/{nome_plugin}/uninstall.sql`. Nesse caso, o método `uninstall` executará apenas as instruções desse arquivo.
