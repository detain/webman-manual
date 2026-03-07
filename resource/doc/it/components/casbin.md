# Casbin

## Descrizione

Casbin è un framework open source potente ed efficiente per il controllo degli accessi. Il suo meccanismo di gestione delle autorizzazioni supporta più modelli di controllo degli accessi.

## Indirizzo del progetto

https://github.com/teamones-open/casbin

## Installazione

```php
composer require teamones/casbin
```

## Sito web ufficiale Casbin

Per un utilizzo dettagliato, consultare la documentazione ufficiale in cinese. Questo documento illustra solo come configurare e utilizzare Casbin in webman.

https://casbin.org/docs/zh-CN/overview

## Struttura delle directory

```
.
├── config                        Directory di configurazione
│   ├── casbin-restful-model.conf File di configurazione del modello di autorizzazioni
│   ├── casbin.php                Configurazione Casbin
......
├── database                      File del database
│   ├── migrations                File di migrazione
│   │   └── 20210218074218_create_rule_table.php
......
```

## File di migrazione del database

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
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'Tabella delle regole']);

        // Aggiungere campi dati
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => 'ID chiave primaria'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'Tipo di regola'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // Eseguire la creazione
        $table->create();
    }
}
```

## Configurazione Casbin

Per la sintassi di configurazione del modello di regole di autorizzazione, consultare: https://casbin.org/docs/zh-CN/syntax-for-models

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // File di configurazione del modello di regole di autorizzazione
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // È possibile configurare più modelli di autorizzazione
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // File di configurazione del modello di regole di autorizzazione
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\RBACRule::class,
        ],
    ],
];
```

### Adattatore

Il pacchetto Composer attuale è adattato ai metodi model di think-orm. Per altri ORM, consultare vendor/teamones/src/adapters/DatabaseAdapter.php

Quindi modificare la configurazione:

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // File di configurazione del modello di regole di autorizzazione
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // Configurare qui il tipo in modalità adattatore
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## Istruzioni per l'uso

### Importazione

```php
# Importazione
use teamones\casbin\Enforcer;
```

### Due modalità d'uso

```php
# 1. Utilizzo della configurazione predefinita
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. Utilizzo della configurazione rbac personalizzata
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### Introduzione alle API comuni

Per maggiori informazioni sull'uso delle API, consultare la documentazione ufficiale:

- API di gestione: https://casbin.org/docs/zh-CN/management-api
- API RBAC: https://casbin.org/docs/zh-CN/rbac-api

```php
# Aggiungere autorizzazione per un utente

Enforcer::addPermissionForUser('user1', '/user', 'read');

# Rimuovere un'autorizzazione da un utente

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# Ottenere tutte le autorizzazioni di un utente

Enforcer::getPermissionsForUser('user1');

# Aggiungere un ruolo per un utente

Enforcer::addRoleForUser('user1', 'role1');

# Aggiungere autorizzazione per un ruolo

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# Ottenere tutti i ruoli

Enforcer::getAllRoles();

# Ottenere tutti i ruoli di un utente

Enforcer::getRolesForUser('user1');

# Ottenere gli utenti in base al ruolo

Enforcer::getUsersForRole('role1');

# Verificare se un utente appartiene a un ruolo

Enforcer::hasRoleForUser('user1', 'role1');

# Rimuovere il ruolo di un utente

Enforcer::deleteRoleForUser('user1', 'role1');

# Rimuovere tutti i ruoli di un utente

Enforcer::deleteRolesForUser('user1');

# Rimuovere un ruolo

Enforcer::deleteRole('role1');

# Rimuovere un'autorizzazione

Enforcer::deletePermission('/user', 'read');

# Rimuovere tutte le autorizzazioni di un utente o di un ruolo

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# Verificare autorizzazione, restituisce true o false

Enforcer::enforce("user1", "/user", "edit");
```
