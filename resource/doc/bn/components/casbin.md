# Casbin

## বর্ণনা

Casbin একটি শক্তিশালী এবং দক্ষ ওপেন সোর্স অ্যাক্সেস কন্ট্রোল ফ্রেমওয়ার্ক। এর অনুমতি ব্যবস্থাপনা প্রক্রিয়া বিভিন্ন অ্যাক্সেস কন্ট্রোল মডেল সমর্থন করে।

## প্রজেক্ট ঠিকানা

https://github.com/teamones-open/casbin

## ইনস্টলেশন

```php
composer require teamones/casbin
```

## Casbin অফিসিয়াল ওয়েবসাইট

বিস্তারিত ব্যবহারের জন্য অফিসিয়াল চীনা ডকুমেন্টেশনে যান। এখানে শুধুমাত্র webman এ Casbin কনফিগার এবং ব্যবহার করার পদ্ধতি বর্ণিত হয়েছে।

https://casbin.org/docs/zh-CN/overview

## ডিরেক্টরি স্ট্রাকচার

```
.
├── config                        কনফিগারেশন ডিরেক্টরি
│   ├── casbin-restful-model.conf অনুমতি মডেল কনফিগারেশন ফাইল
│   ├── casbin.php                Casbin কনফিগারেশন
......
├── database                      ডেটাবেস ফাইল
│   ├── migrations                মাইগ্রেশন ফাইল
│   │   └── 20210218074218_create_rule_table.php
......
```

## ডেটাবেস মাইগ্রেশন ফাইল

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
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'নিয়ম টেবিল']);

        // ডেটা ফিল্ড যোগ করুন
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => 'প্রাইমারি কী ID'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'নিয়ম ধরন'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // তৈরি সম্পাদন করুন
        $table->create();
    }
}
```

## Casbin কনফিগারেশন

অনুমতি নিয়ম মডেল কনফিগারেশন সিনট্যাক্সের জন্য দেখুন: https://casbin.org/docs/zh-CN/syntax-for-models

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // অনুমতি নিয়ম মডেল কনফিগারেশন ফাইল
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // একাধিক অনুমতি মডেল কনফিগার করা যেতে পারে
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // অনুমতি নিয়ম মডেল কনফিগারেশন ফাইল
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\RBACRule::class,
        ],
    ],
];
```

### অ্যাডাপ্টার

বর্তমান Composer প্যাকেজ think-orm এর model মেথডের সাথে উপযুক্ত। অন্যান্য ORM এর জন্য vendor/teamones/src/adapters/DatabaseAdapter.php দেখুন

তারপর কনফিগারেশন পরিবর্তন করুন:

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // অনুমতি নিয়ম মডেল কনফিগারেশন ফাইল
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // এখানে টাইপ অ্যাডাপ্টার মোড হিসাবে কনফিগার করুন
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## ব্যবহার নির্দেশিকা

### ইমপোর্ট

```php
# ইমপোর্ট
use teamones\casbin\Enforcer;
```

### দুটি ব্যবহার পদ্ধতি

```php
# 1. ডিফল্ট কনফিগারেশন ব্যবহার করুন
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. কাস্টম rbac কনফিগারেশন ব্যবহার করুন
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### সাধারণ API পরিচয়

আরও API ব্যবহারের জন্য অফিসিয়াল ডকুমেন্টেশন দেখুন:

- Management API: https://casbin.org/docs/zh-CN/management-api
- RBAC API: https://casbin.org/docs/zh-CN/rbac-api

```php
# ব্যবহারকারীর জন্য অনুমতি যোগ করুন

Enforcer::addPermissionForUser('user1', '/user', 'read');

# ব্যবহারকারীর অনুমতি মুছুন

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# ব্যবহারকারীর সব অনুমতি পান

Enforcer::getPermissionsForUser('user1');

# ব্যবহারকারীর জন্য ভূমিকা যোগ করুন

Enforcer::addRoleForUser('user1', 'role1');

# ভূমিকার জন্য অনুমতি যোগ করুন

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# সব ভূমিকা পান

Enforcer::getAllRoles();

# ব্যবহারকারীর সব ভূমিকা পান

Enforcer::getRolesForUser('user1');

# ভূমিকা অনুযায়ী ব্যবহারকারী পান

Enforcer::getUsersForRole('role1');

# ব্যবহারকারী ভূমিকার অন্তর্ভুক্ত কিনা পরীক্ষা করুন

Enforcer::hasRoleForUser('user1', 'role1');

# ব্যবহারকারীর ভূমিকা মুছুন

Enforcer::deleteRoleForUser('user1', 'role1');

# ব্যবহারকারীর সব ভূমিকা মুছুন

Enforcer::deleteRolesForUser('user1');

# ভূমিকা মুছুন

Enforcer::deleteRole('role1');

# অনুমতি মুছুন

Enforcer::deletePermission('/user', 'read');

# ব্যবহারকারী বা ভূমিকার সব অনুমতি মুছুন

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# অনুমতি পরীক্ষা করুন, true বা false ফেরত দেয়

Enforcer::enforce("user1", "/user", "edit");
```
