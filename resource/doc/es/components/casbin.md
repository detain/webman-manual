# Casbin

## Descripción

Casbin es un potente y eficiente framework de control de acceso de código abierto. Su mecanismo de gestión de permisos admite múltiples modelos de control de acceso.

## Dirección del proyecto

https://github.com/teamones-open/casbin

## Instalación

```php
composer require teamones/casbin
```

## Sitio web oficial de Casbin

Para un uso detallado, consulte la documentación oficial en chino. Este documento solo explica cómo configurar y usar Casbin en webman.

https://casbin.org/docs/zh-CN/overview

## Estructura de directorios

```
.
├── config                        Directorio de configuración
│   ├── casbin-restful-model.conf Archivo de configuración del modelo de permisos
│   ├── casbin.php                Configuración de Casbin
......
├── database                      Archivos de base de datos
│   ├── migrations                Archivos de migración
│   │   └── 20210218074218_create_rule_table.php
......
```

## Archivo de migración de base de datos

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
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'Tabla de reglas']);

        // Añadir campos de datos
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => 'ID de clave primaria'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'Tipo de regla'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // Ejecutar la creación
        $table->create();
    }
}
```

## Configuración de Casbin

Para la sintaxis de configuración del modelo de reglas de permisos, consulte: https://casbin.org/docs/zh-CN/syntax-for-models

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // Archivo de configuración del modelo de reglas de permisos
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // Se pueden configurar múltiples modelos de permisos
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // Archivo de configuración del modelo de reglas de permisos
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

El paquete actual está adaptado a los métodos model de think-orm. Para otros ORM, consulte vendor/teamones/src/adapters/DatabaseAdapter.php

Luego modifique la configuración:

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // Archivo de configuración del modelo de reglas de permisos
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // Configurar el tipo en modo adaptador aquí
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## Instrucciones de uso

### Importar

```php
# Importar
use teamones\casbin\Enforcer;
```

### Dos formas de uso

```php
# 1. Usar la configuración por defecto
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. Usar configuración rbac personalizada
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### Introducción a las API comunes

Para más información sobre las API, consulte la documentación oficial:

- API de gestión: https://casbin.org/docs/zh-CN/management-api
- API RBAC: https://casbin.org/docs/zh-CN/rbac-api

```php
# Añadir permiso para un usuario

Enforcer::addPermissionForUser('user1', '/user', 'read');

# Eliminar un permiso de un usuario

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# Obtener todos los permisos de un usuario

Enforcer::getPermissionsForUser('user1');

# Añadir rol para un usuario

Enforcer::addRoleForUser('user1', 'role1');

# Añadir permiso para un rol

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# Obtener todos los roles

Enforcer::getAllRoles();

# Obtener todos los roles de un usuario

Enforcer::getRolesForUser('user1');

# Obtener usuarios por rol

Enforcer::getUsersForRole('role1');

# Comprobar si un usuario pertenece a un rol

Enforcer::hasRoleForUser('user1', 'role1');

# Eliminar rol de un usuario

Enforcer::deleteRoleForUser('user1', 'role1');

# Eliminar todos los roles de un usuario

Enforcer::deleteRolesForUser('user1');

# Eliminar rol

Enforcer::deleteRole('role1');

# Eliminar permiso

Enforcer::deletePermission('/user', 'read');

# Eliminar todos los permisos de un usuario o rol

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# Comprobar permiso, devuelve true o false

Enforcer::enforce("user1", "/user", "edit");
```
