# Casbin

## Description

Casbin est un framework open source puissant et performant de contrôle d'accès. Son mécanisme de gestion des permissions prend en charge plusieurs modèles de contrôle d'accès.

## Adresse du projet

https://github.com/teamones-open/casbin

## Installation

```php
composer require teamones/casbin
```

## Site officiel Casbin

Pour une utilisation détaillée, veuillez consulter la documentation officielle en chinois. Ce document ne traite que de la configuration et de l'utilisation de Casbin dans webman.

https://casbin.org/docs/zh-CN/overview

## Structure du répertoire

```
.
├── config                        Répertoire de configuration
│   ├── casbin-restful-model.conf Fichier de configuration du modèle de permissions
│   ├── casbin.php                Configuration Casbin
......
├── database                      Fichiers de base de données
│   ├── migrations                Fichiers de migration
│   │   └── 20210218074218_create_rule_table.php
......
```

## Fichier de migration de la base de données

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
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'Table des règles']);

        // Ajouter les champs de données
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => 'ID de clé primaire'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'Type de règle'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // Exécuter la création
        $table->create();
    }
}
```

## Configuration Casbin

Pour la syntaxe de configuration du modèle de règles de permissions, voir : https://casbin.org/docs/zh-CN/syntax-for-models

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // Fichier de configuration du modèle de règles de permissions
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // Plusieurs modèles de permissions peuvent être configurés
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // Fichier de configuration du modèle de règles de permissions
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\RBACRule::class,
        ],
    ],
];
```

### Adaptateur

Le package actuel est adapté aux méthodes model de think-orm. Pour les autres ORM, consulter vendor/teamones/src/adapters/DatabaseAdapter.php

Puis modifier la configuration :

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // Fichier de configuration du modèle de règles de permissions
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // Configurer le type en mode adaptateur ici
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## Instructions d'utilisation

### Import

```php
# Import
use teamones\casbin\Enforcer;
```

### Deux modes d'utilisation

```php
# 1. Utiliser la configuration par défaut
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. Utiliser une configuration rbac personnalisée
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### Introduction aux API courantes

Pour plus d'informations sur les API, consulter la documentation officielle :

- API de gestion : https://casbin.org/docs/zh-CN/management-api
- API RBAC : https://casbin.org/docs/zh-CN/rbac-api

```php
# Ajouter une permission pour un utilisateur

Enforcer::addPermissionForUser('user1', '/user', 'read');

# Supprimer une permission d'un utilisateur

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# Obtenir toutes les permissions d'un utilisateur

Enforcer::getPermissionsForUser('user1');

# Ajouter un rôle pour un utilisateur

Enforcer::addRoleForUser('user1', 'role1');

# Ajouter une permission pour un rôle

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# Obtenir tous les rôles

Enforcer::getAllRoles();

# Obtenir tous les rôles d'un utilisateur

Enforcer::getRolesForUser('user1');

# Obtenir les utilisateurs par rôle

Enforcer::getUsersForRole('role1');

# Vérifier si un utilisateur appartient à un rôle

Enforcer::hasRoleForUser('user1', 'role1');

# Supprimer le rôle d'un utilisateur

Enforcer::deleteRoleForUser('user1', 'role1');

# Supprimer tous les rôles d'un utilisateur

Enforcer::deleteRolesForUser('user1');

# Supprimer un rôle

Enforcer::deleteRole('role1');

# Supprimer une permission

Enforcer::deletePermission('/user', 'read');

# Supprimer toutes les permissions d'un utilisateur ou d'un rôle

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# Vérifier une permission, retourne true ou false

Enforcer::enforce("user1", "/user", "edit");
```
