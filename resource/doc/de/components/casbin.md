# Casbin

## Beschreibung

Casbin ist ein leistungsfähiges und effizientes Open-Source-Zugriffskontroll-Framework. Sein Berechtigungsverwaltungsmechanismus unterstützt mehrere Zugriffskontrollmodelle.

## Projektadresse

https://github.com/teamones-open/casbin

## Installation

```php
composer require teamones/casbin
```

## Casbin-Website

Für ausführliche Anleitungen konsultieren Sie bitte die offizielle chinesische Dokumentation. Hier wird nur die Konfiguration und Verwendung in webman erläutert.

https://casbin.org/docs/zh-CN/overview

## Verzeichnisstruktur

```
.
├── config                        Konfigurationsverzeichnis
│   ├── casbin-restful-model.conf Konfigurationsdatei des Berechtigungsmodells
│   ├── casbin.php                Casbin-Konfiguration
......
├── database                      Datenbankdateien
│   ├── migrations                Migrationsdateien
│   │   └── 20210218074218_create_rule_table.php
......
```

## Datenbank-Migrationsdatei

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
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'Regeltabelle']);

        // Datenfelder hinzufügen
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => 'Primärschlüssel-ID'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'Regeltyp'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // Erstellung ausführen
        $table->create();
    }
}
```

## Casbin-Konfiguration

Zur Syntax der Berechtigungsregelmodell-Konfiguration siehe: https://casbin.org/docs/zh-CN/syntax-for-models

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // Berechtigungsregelmodell-Konfigurationsdatei
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // Mehrere Berechtigungsmodelle können konfiguriert werden
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // Berechtigungsregelmodell-Konfigurationsdatei
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\RBACRule::class,
        ],
    ],
];
```

### Adapter

Die aktuelle Composer-Paket-Implementierung ist an die Model-Methoden von think-orm angepasst. Für andere ORMs siehe vendor/teamones/src/adapters/DatabaseAdapter.php

Anschließend die Konfiguration anpassen:

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // Berechtigungsregelmodell-Konfigurationsdatei
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // Hier den Typ als Adapter-Modus konfigurieren
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## Verwendungsanleitung

### Import

```php
# Import
use teamones\casbin\Enforcer;
```

### Zwei Verwendungsarten

```php
# 1. Standardkonfiguration verwenden
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. Benutzerdefinierte rbac-Konfiguration verwenden
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### Übersicht gängiger APIs

Weitere API-Verwendung siehe offizielle Dokumentation:

- Management API: https://casbin.org/docs/zh-CN/management-api
- RBAC API: https://casbin.org/docs/zh-CN/rbac-api

```php
# Berechtigung für Benutzer hinzufügen

Enforcer::addPermissionForUser('user1', '/user', 'read');

# Berechtigung für Benutzer entfernen

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# Alle Berechtigungen eines Benutzers abrufen

Enforcer::getPermissionsForUser('user1');

# Rolle für Benutzer hinzufügen

Enforcer::addRoleForUser('user1', 'role1');

# Berechtigung für Rolle hinzufügen

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# Alle Rollen abrufen

Enforcer::getAllRoles();

# Alle Rollen eines Benutzers abrufen

Enforcer::getRolesForUser('user1');

# Benutzer nach Rolle abrufen

Enforcer::getUsersForRole('role1');

# Prüfen, ob Benutzer einer Rolle angehört

Enforcer::hasRoleForUser('user1', 'role1');

# Benutzerrolle entfernen

Enforcer::deleteRoleForUser('user1', 'role1');

# Alle Rollen eines Benutzers entfernen

Enforcer::deleteRolesForUser('user1');

# Rolle entfernen

Enforcer::deleteRole('role1');

# Berechtigung entfernen

Enforcer::deletePermission('/user', 'read');

# Alle Berechtigungen eines Benutzers oder einer Rolle entfernen

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# Berechtigung prüfen, gibt true oder false zurück

Enforcer::enforce("user1", "/user", "edit");
```
