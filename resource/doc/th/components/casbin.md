# Casbin

## คำอธิบาย

Casbin คือเฟรมเวิร์กการควบคุมการเข้าถึงโอเพนซอร์สที่ทรงพลังและมีประสิทธิภาพ กลไกการจัดการสิทธิ์ของมันรองรับโมเดลการควบคุมการเข้าถึงหลากหลายแบบ

## ที่อยู่โปรเจกต์

https://github.com/teamones-open/casbin

## การติดตั้ง

```php
composer require teamones/casbin
```

## เว็บไซต์อย่างเป็นทางการของ Casbin

สำหรับการใช้งานโดยละเอียด โปรดดูเอกสารจีนอย่างเป็นทางการ เอกสารนี้เน้นเฉพาะการตั้งค่าและใช้งาน Casbin ใน webman

https://casbin.org/docs/zh-CN/overview

## โครงสร้างไดเรกทอรี

```
.
├── config                        ไดเรกทอรีการกำหนดค่า
│   ├── casbin-restful-model.conf ไฟล์กำหนดค่าโมเดลสิทธิ์ที่ใช้
│   ├── casbin.php                การกำหนดค่า Casbin
......
├── database                      ไฟล์ฐานข้อมูล
│   ├── migrations                ไฟล์การโยกย้าย
│   │   └── 20210218074218_create_rule_table.php
......
```

## ไฟล์การโยกย้ายฐานข้อมูล

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
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'ตารางกฎ']);

        // เพิ่มฟิลด์ข้อมูล
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => 'ID คีย์หลัก'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'ประเภทกฎ'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // ดำเนินการสร้าง
        $table->create();
    }
}
```

## การกำหนดค่า Casbin

สำหรับไวยากรณ์การกำหนดค่าโมเดลกฎสิทธิ์ ดูที่: https://casbin.org/docs/zh-CN/syntax-for-models

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // ไฟล์กำหนดค่าโมเดลกฎสิทธิ์
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // สามารถกำหนดค่าโมเดลสิทธิ์ได้หลายแบบ
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // ไฟล์กำหนดค่าโมเดลกฎสิทธิ์
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\RBACRule::class,
        ],
    ],
];
```

### ตัวปรับ

แพ็กเกจ Composer ปัจจุบันปรับให้เข้ากับเมธอด model ของ think-orm สำหรับ ORM อื่น ดูที่ vendor/teamones/src/adapters/DatabaseAdapter.php

จากนั้นแก้ไขการกำหนดค่า:

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // ไฟล์กำหนดค่าโมเดลกฎสิทธิ์
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // กำหนดประเภทเป็นโหมดตัวปรับที่นี่
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## คำแนะนำการใช้งาน

### การนำเข้า

```php
# การนำเข้า
use teamones\casbin\Enforcer;
```

### สองวิธีการใช้งาน

```php
# 1. ใช้การกำหนดค่าเริ่มต้น
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. ใช้การกำหนดค่า rbac ที่กำหนดเอง
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### บทนำ API ที่ใช้บ่อย

สำหรับการใช้ API เพิ่มเติม ดูเอกสารอย่างเป็นทางการ:

- Management API: https://casbin.org/docs/zh-CN/management-api
- RBAC API: https://casbin.org/docs/zh-CN/rbac-api

```php
# เพิ่มสิทธิ์ให้ผู้ใช้

Enforcer::addPermissionForUser('user1', '/user', 'read');

# ลบสิทธิ์ของผู้ใช้

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# รับสิทธิ์ทั้งหมดของผู้ใช้

Enforcer::getPermissionsForUser('user1');

# เพิ่มบทบาทให้ผู้ใช้

Enforcer::addRoleForUser('user1', 'role1');

# เพิ่มสิทธิ์ให้บทบาท

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# รับบทบาททั้งหมด

Enforcer::getAllRoles();

# รับบทบาททั้งหมดของผู้ใช้

Enforcer::getRolesForUser('user1');

# รับผู้ใช้ตามบทบาท

Enforcer::getUsersForRole('role1');

# ตรวจสอบว่าผู้ใช้อยู่ในบทบาทหรือไม่

Enforcer::hasRoleForUser('user1', 'role1');

# ลบบทบาทของผู้ใช้

Enforcer::deleteRoleForUser('user1', 'role1');

# ลบบทบาททั้งหมดของผู้ใช้

Enforcer::deleteRolesForUser('user1');

# ลบบทบาท

Enforcer::deleteRole('role1');

# ลบสิทธิ์

Enforcer::deletePermission('/user', 'read');

# ลบสิทธิ์ทั้งหมดของผู้ใช้หรือบทบาท

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# ตรวจสอบสิทธิ์ คืนค่า true หรือ false

Enforcer::enforce("user1", "/user", "edit");
```
