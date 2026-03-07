# Casbin

## Descrição

Casbin é um framework de controle de acesso open source poderoso e eficiente. Seu mecanismo de gerenciamento de permissões suporta diversos modelos de controle de acesso.

## Endereço do projeto

https://github.com/teamones-open/casbin

## Instalação

```php
composer require teamones/casbin
```

## Site oficial do Casbin

Para uso detalhado, consulte a documentação oficial em chinês. Este documento aborda apenas como configurar e usar o Casbin no webman.

https://casbin.org/docs/zh-CN/overview

## Estrutura do diretório

```
.
├── config                        Diretório de configuração
│   ├── casbin-restful-model.conf Arquivo de configuração do modelo de permissões
│   ├── casbin.php                Configuração do Casbin
......
├── database                      Arquivos de banco de dados
│   ├── migrations                Arquivos de migração
│   │   └── 20210218074218_create_rule_table.php
......
```

## Arquivo de migração do banco de dados

```php
<?php

use Phinx\Migration\AbstractMigration;

class CreateRuleTable extends AbstractMigration
{
    /**
     * Change Method.
     *
     * Write your reversible migrations using this method.
     *
     * More information on writing migrations is available here:
     * http://docs.phinx.org/en/latest/migrations.html#the-abstractmigration-class
     *
     * The following commands can be used in this method and Phinx will
     * automatically reverse them when rolling back:
     *
     *    createTable
     *    renameTable
     *    addColumn
     *    addCustomColumn
     *    renameColumn
     *    addIndex
     *    addForeignKey
     *
     * Any other destructive changes will result in an error when trying to
     * rollback the migration.
     *
     * Remember to call "create()" or "update()" and NOT "save()" when working
     * with the Table class.
     */
    public function change()
    {
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'Tabela de regras']);

        // Adicionar campos de dados
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => 'ID da chave primária'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'Tipo de regra'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // Executar a criação
        $table->create();
    }
}
```

## Configuração do Casbin

Para a sintaxe de configuração do modelo de regras de permissões, consulte: https://casbin.org/docs/zh-CN/syntax-for-models

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // Arquivo de configuração do modelo de regras de permissões
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // Múltiplos modelos de permissões podem ser configurados
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // Arquivo de configuração do modelo de regras de permissões
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\RBACRule::class,
        ],
    ],
];
```

### Adaptador

O pacote Composer atual está adaptado aos métodos model do think-orm. Para outros ORMs, consulte vendor/teamones/src/adapters/DatabaseAdapter.php

Em seguida, altere a configuração:

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // Arquivo de configuração do modelo de regras de permissões
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // Configurar o tipo como modo adaptador aqui
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## Instruções de uso

### Importação

```php
# Importação
use teamones\casbin\Enforcer;
```

### Dois métodos de uso

```php
# 1. Usar a configuração padrão
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. Usar configuração rbac personalizada
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### Introdução às APIs comuns

Para mais informações sobre as APIs, consulte a documentação oficial:

- API de gerenciamento: https://casbin.org/docs/zh-CN/management-api
- API RBAC: https://casbin.org/docs/zh-CN/rbac-api

```php
# Adicionar permissão para usuário

Enforcer::addPermissionForUser('user1', '/user', 'read');

# Excluir permissão de um usuário

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# Obter todas as permissões de um usuário

Enforcer::getPermissionsForUser('user1');

# Adicionar função para usuário

Enforcer::addRoleForUser('user1', 'role1');

# Adicionar permissão para função

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# Obter todas as funções

Enforcer::getAllRoles();

# Obter todas as funções de um usuário

Enforcer::getRolesForUser('user1');

# Obter usuários por função

Enforcer::getUsersForRole('role1');

# Verificar se o usuário pertence a uma função

Enforcer::hasRoleForUser('user1', 'role1');

# Excluir função do usuário

Enforcer::deleteRoleForUser('user1', 'role1');

# Excluir todas as funções do usuário

Enforcer::deleteRolesForUser('user1');

# Excluir função

Enforcer::deleteRole('role1');

# Excluir permissão

Enforcer::deletePermission('/user', 'read');

# Excluir todas as permissões do usuário ou função

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# Verificar permissão, retorna true ou false

Enforcer::enforce("user1", "/user", "edit");
```
