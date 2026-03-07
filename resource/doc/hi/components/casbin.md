# Casbin

## विवरण

Casbin एक शक्तिशाली और कुशल ओपन सोर्स एक्सेस कंट्रोल फ्रेमवर्क है। इसकी अनुमति प्रबंधन व्यवस्था कई एक्सेस कंट्रोल मॉडल का समर्थन करती है।

## प्रोजेक्ट पता

https://github.com/teamones-open/casbin

## स्थापना

```php
composer require teamones/casbin
```

## Casbin आधिकारिक वेबसाइट

विस्तृत उपयोग के लिए आधिकारिक चीनी दस्तावेज़ देखें। यह दस्तावेज़ केवल webman में Casbin को कैसे कॉन्फ़िगर और उपयोग करना है यह बताता है।

https://casbin.org/docs/zh-CN/overview

## निर्देशिका संरचना

```
.
├── config                        कॉन्फ़िगरेशन निर्देशिका
│   ├── casbin-restful-model.conf अनुमति मॉडल कॉन्फ़िगरेशन फ़ाइल
│   ├── casbin.php                Casbin कॉन्फ़िगरेशन
......
├── database                      डेटाबेस फ़ाइलें
│   ├── migrations                माइग्रेशन फ़ाइलें
│   │   └── 20210218074218_create_rule_table.php
......
```

## डेटाबेस माइग्रेशन फ़ाइल

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
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'नियम तालिका']);

        // डेटा फ़ील्ड जोड़ें
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => 'प्राथमिक कुंजी ID'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'नियम प्रकार'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // निर्माण निष्पादित करें
        $table->create();
    }
}
```

## Casbin कॉन्फ़िगरेशन

अनुमति नियम मॉडल कॉन्फ़िगरेशन व्याकरण के लिए देखें: https://casbin.org/docs/zh-CN/syntax-for-models

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // अनुमति नियम मॉडल कॉन्फ़िगरेशन फ़ाइल
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // कई अनुमति मॉडल कॉन्फ़िगर किए जा सकते हैं
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // अनुमति नियम मॉडल कॉन्फ़िगरेशन फ़ाइल
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\RBACRule::class,
        ],
    ],
];
```

### अडैप्टर

वर्तमान Composer पैकेज think-orm के model विधियों के लिए अनुकूलित है। अन्य ORM के लिए vendor/teamones/src/adapters/DatabaseAdapter.php देखें

फिर कॉन्फ़िगरेशन संशोधित करें:

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // अनुमति नियम मॉडल कॉन्फ़िगरेशन फ़ाइल
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // यहां प्रकार को अडैप्टर मोड के रूप में कॉन्फ़िगर करें
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## उपयोग निर्देश

### इम्पोर्ट

```php
# इम्पोर्ट
use teamones\casbin\Enforcer;
```

### दो उपयोग विधियाँ

```php
# 1. डिफ़ॉल्ट कॉन्फ़िगरेशन का उपयोग करें
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. कस्टम rbac कॉन्फ़िगरेशन का उपयोग करें
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### सामान्य API परिचय

अधिक API उपयोग के लिए आधिकारिक दस्तावेज़ देखें:

- Management API: https://casbin.org/docs/zh-CN/management-api
- RBAC API: https://casbin.org/docs/zh-CN/rbac-api

```php
# उपयोगकर्ता के लिए अनुमति जोड़ें

Enforcer::addPermissionForUser('user1', '/user', 'read');

# उपयोगकर्ता की अनुमति हटाएं

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# उपयोगकर्ता की सभी अनुमतियाँ प्राप्त करें

Enforcer::getPermissionsForUser('user1');

# उपयोगकर्ता के लिए भूमिका जोड़ें

Enforcer::addRoleForUser('user1', 'role1');

# भूमिका के लिए अनुमति जोड़ें

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# सभी भूमिकाएँ प्राप्त करें

Enforcer::getAllRoles();

# उपयोगकर्ता की सभी भूमिकाएँ प्राप्त करें

Enforcer::getRolesForUser('user1');

# भूमिका के अनुसार उपयोगकर्ता प्राप्त करें

Enforcer::getUsersForRole('role1');

# जाँचें कि उपयोगकर्ता भूमिका से संबंधित है या नहीं

Enforcer::hasRoleForUser('user1', 'role1');

# उपयोगकर्ता की भूमिका हटाएं

Enforcer::deleteRoleForUser('user1', 'role1');

# उपयोगकर्ता की सभी भूमिकाएँ हटाएं

Enforcer::deleteRolesForUser('user1');

# भूमिका हटाएं

Enforcer::deleteRole('role1');

# अनुमति हटाएं

Enforcer::deletePermission('/user', 'read');

# उपयोगकर्ता या भूमिका की सभी अनुमतियाँ हटाएं

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# अनुमति जाँचें, true या false लौटाता है

Enforcer::enforce("user1", "/user", "edit");
```
