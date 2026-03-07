# Casbin

## Tanım

Casbin, güçlü ve verimli bir açık kaynak erişim kontrol çerçevesidir. İzin yönetim mekanizması birden fazla erişim kontrol modelini destekler.

## Proje Adresi

https://github.com/teamones-open/casbin

## Kurulum

```php
composer require teamones/casbin
```

## Casbin Resmi Web Sitesi

Ayrıntılı kullanım için lütfen resmi Çince belgelere bakın. Bu belge yalnızca webman'da Casbin'in nasıl yapılandırılacağını ve kullanılacağını açıklar.

https://casbin.org/docs/zh-CN/overview

## Dizin Yapısı

```
.
├── config                        Yapılandırma dizini
│   ├── casbin-restful-model.conf İzin modeli yapılandırma dosyası
│   ├── casbin.php                Casbin yapılandırması
......
├── database                      Veritabanı dosyaları
│   ├── migrations                Göç dosyaları
│   │   └── 20210218074218_create_rule_table.php
......
```

## Veritabanı Göç Dosyası

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
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'Kural tablosu']);

        // Veri alanları ekle
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => 'Birincil anahtar ID'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'Kural türü'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // Oluşturmayı yürüt
        $table->create();
    }
}
```

## Casbin Yapılandırması

İzin kuralı modeli yapılandırma sözdizimi için bkz.: https://casbin.org/docs/zh-CN/syntax-for-models

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // İzin kuralı modeli yapılandırma dosyası
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // Birden fazla izin modeli yapılandırılabilir
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // İzin kuralı modeli yapılandırma dosyası
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\RBACRule::class,
        ],
    ],
];
```

### Uyarlayıcı

Mevcut Composer paketi think-orm'un model yöntemlerine uyarlanmıştır. Diğer ORM'ler için vendor/teamones/src/adapters/DatabaseAdapter.php dosyasına bakın

Ardından yapılandırmayı değiştirin:

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // İzin kuralı modeli yapılandırma dosyası
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // Burada türü uyarlayıcı modu olarak yapılandırın
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## Kullanım Talimatları

### İçe Aktarma

```php
# İçe aktarma
use teamones\casbin\Enforcer;
```

### İki Kullanım Yöntemi

```php
# 1. Varsayılan yapılandırmayı kullan
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. Özel rbac yapılandırmasını kullan
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### Yaygın API Girişi

Daha fazla API kullanımı için resmi belgelere bakın:

- Management API: https://casbin.org/docs/zh-CN/management-api
- RBAC API: https://casbin.org/docs/zh-CN/rbac-api

```php
# Kullanıcıya izin ekle

Enforcer::addPermissionForUser('user1', '/user', 'read');

# Kullanıcının iznini sil

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# Kullanıcının tüm izinlerini al

Enforcer::getPermissionsForUser('user1');

# Kullanıcıya rol ekle

Enforcer::addRoleForUser('user1', 'role1');

# Role izin ekle

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# Tüm rolleri al

Enforcer::getAllRoles();

# Kullanıcının tüm rollerini al

Enforcer::getRolesForUser('user1');

# Role göre kullanıcıları al

Enforcer::getUsersForRole('role1');

# Kullanıcının bir role ait olup olmadığını kontrol et

Enforcer::hasRoleForUser('user1', 'role1');

# Kullanıcının rolünü sil

Enforcer::deleteRoleForUser('user1', 'role1');

# Kullanıcının tüm rollerini sil

Enforcer::deleteRolesForUser('user1');

# Rolü sil

Enforcer::deleteRole('role1');

# İzni sil

Enforcer::deletePermission('/user', 'read');

# Kullanıcı veya rolün tüm izinlerini sil

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# İzin kontrolü, true veya false döndürür

Enforcer::enforce("user1", "/user", "edit");
```
