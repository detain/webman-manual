# Casbin

## Описание

Casbin — это мощный и эффективный фреймворк управления доступом с открытым исходным кодом. Его механизм управления правами поддерживает различные модели контроля доступа.

## Адрес проекта

https://github.com/teamones-open/casbin

## Установка

```php
composer require teamones/casbin
```

## Официальный сайт Casbin

Для подробного использования обратитесь к официальной китайской документации. Здесь описывается только настройка и использование Casbin в webman.

https://casbin.org/docs/zh-CN/overview

## Структура каталогов

```
.
├── config                        Каталог конфигурации
│   ├── casbin-restful-model.conf Файл конфигурации модели разрешений
│   ├── casbin.php                Конфигурация Casbin
......
├── database                      Файлы базы данных
│   ├── migrations                Файлы миграции
│   │   └── 20210218074218_create_rule_table.php
......
```

## Файл миграции базы данных

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
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'Таблица правил']);

        // Добавление полей данных
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => 'ID первичного ключа'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'Тип правила'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // Выполнение создания
        $table->create();
    }
}
```

## Конфигурация Casbin

Синтаксис конфигурации модели правил разрешений см.: https://casbin.org/docs/zh-CN/syntax-for-models

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // Файл конфигурации модели правил разрешений
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // Можно настроить несколько моделей разрешений
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // Файл конфигурации модели правил разрешений
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\RBACRule::class,
        ],
    ],
];
```

### Адаптер

Текущий пакет Composer адаптирован к методам model think-orm. Для других ORM см. vendor/teamones/src/adapters/DatabaseAdapter.php

Затем измените конфигурацию:

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // Файл конфигурации модели правил разрешений
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // Здесь тип настраивается в режиме адаптера
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## Инструкция по использованию

### Импорт

```php
# Импорт
use teamones\casbin\Enforcer;
```

### Два способа использования

```php
# 1. Использование конфигурации по умолчанию
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. Использование пользовательской конфигурации rbac
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### Обзор распространённых API

Для подробной информации по API обратитесь к официальной документации:

- API управления: https://casbin.org/docs/zh-CN/management-api
- RBAC API: https://casbin.org/docs/zh-CN/rbac-api

```php
# Добавить разрешение для пользователя

Enforcer::addPermissionForUser('user1', '/user', 'read');

# Удалить разрешение у пользователя

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# Получить все разрешения пользователя

Enforcer::getPermissionsForUser('user1');

# Добавить роль для пользователя

Enforcer::addRoleForUser('user1', 'role1');

# Добавить разрешение для роли

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# Получить все роли

Enforcer::getAllRoles();

# Получить все роли пользователя

Enforcer::getRolesForUser('user1');

# Получить пользователей по роли

Enforcer::getUsersForRole('role1');

# Проверить, принадлежит ли пользователь роли

Enforcer::hasRoleForUser('user1', 'role1');

# Удалить роль пользователя

Enforcer::deleteRoleForUser('user1', 'role1');

# Удалить все роли пользователя

Enforcer::deleteRolesForUser('user1');

# Удалить роль

Enforcer::deleteRole('role1');

# Удалить разрешение

Enforcer::deletePermission('/user', 'read');

# Удалить все разрешения пользователя или роли

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# Проверить разрешение, возвращает true или false

Enforcer::enforce("user1", "/user", "edit");
```
