# Casbin

## الوصف

Casbin هو إطار عمل قوي وفعال للتحكم في الوصول مفتوح المصدر. تدعم آلية إدارة الصلاحيات الخاصة به نماذج متعددة للتحكم في الوصول.

## عنوان المشروع

https://github.com/teamones-open/casbin

## التثبيت

```php
composer require teamones/casbin
```

## الموقع الرسمي لـ Casbin

للتفاصيل الكاملة حول الاستخدام، راجع الوثائق الرسمية بالصينية. يشرح هذا المستند فقط كيفية تكوين واستخدام Casbin في webman.

https://casbin.org/docs/zh-CN/overview

## هيكل الدليل

```
.
├── config                        مجلد التكوين
│   ├── casbin-restful-model.conf ملف تكوين نموذج الصلاحيات
│   ├── casbin.php                تكوين Casbin
......
├── database                      ملفات قاعدة البيانات
│   ├── migrations                ملفات الترحيل
│   │   └── 20210218074218_create_rule_table.php
......
```

## ملف ترحيل قاعدة البيانات

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
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'جدول القواعد']);

        // إضافة حقول البيانات
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => 'معرف المفتاح الأساسي'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'نوع القاعدة'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // تنفيذ الإنشاء
        $table->create();
    }
}
```

## تكوين Casbin

لتركيب نموذج قواعد الصلاحيات، راجع: https://casbin.org/docs/zh-CN/syntax-for-models

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // ملف تكوين نموذج قواعد الصلاحيات
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // يمكن تكوين نماذج صلاحيات متعددة
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // ملف تكوين نموذج قواعد الصلاحيات
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\RBACRule::class,
        ],
    ],
];
```

### المحول

الحزمة الحالية متوافقة مع طرائق model الخاصة بـ think-orm. للـ ORM الأخرى، راجع vendor/teamones/src/adapters/DatabaseAdapter.php

ثم عدّل التكوين:

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // ملف تكوين نموذج قواعد الصلاحيات
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // تكوين النوع كنمط محول هنا
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## تعليمات الاستخدام

### الاستيراد

```php
# الاستيراد
use teamones\casbin\Enforcer;
```

### طريقتان للاستخدام

```php
# 1. استخدام التكوين الافتراضي
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. استخدام تكوين rbac المخصص
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### مقدمة لـ API الشائعة

لمزيد من استخدامات API، راجع الوثائق الرسمية:

- Management API: https://casbin.org/docs/zh-CN/management-api
- RBAC API: https://casbin.org/docs/zh-CN/rbac-api

```php
# إضافة صلاحية للمستخدم

Enforcer::addPermissionForUser('user1', '/user', 'read');

# حذف صلاحية المستخدم

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# الحصول على جميع صلاحيات المستخدم

Enforcer::getPermissionsForUser('user1');

# إضافة دور للمستخدم

Enforcer::addRoleForUser('user1', 'role1');

# إضافة صلاحية للدور

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# الحصول على جميع الأدوار

Enforcer::getAllRoles();

# الحصول على جميع أدوار المستخدم

Enforcer::getRolesForUser('user1');

# الحصول على المستخدمين حسب الدور

Enforcer::getUsersForRole('role1');

# التحقق من انتماء المستخدم لدور

Enforcer::hasRoleForUser('user1', 'role1');

# حذف دور المستخدم

Enforcer::deleteRoleForUser('user1', 'role1');

# حذف جميع أدوار المستخدم

Enforcer::deleteRolesForUser('user1');

# حذف الدور

Enforcer::deleteRole('role1');

# حذف الصلاحية

Enforcer::deletePermission('/user', 'read');

# حذف جميع صلاحيات المستخدم أو الدور

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# التحقق من الصلاحية، إرجاع true أو false

Enforcer::enforce("user1", "/user", "edit");
```
