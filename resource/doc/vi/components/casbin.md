# Casbin

## Giới thiệu

Casbin là một framework kiểm soát truy cập mã nguồn mở mạnh mẽ và hiệu quả. Cơ chế quản lý quyền của nó hỗ trợ nhiều mô hình kiểm soát truy cập.

## Địa chỉ dự án

https://github.com/teamones-open/casbin

## Cài đặt

```php
composer require teamones/casbin
```

## Trang web chính thức Casbin

Để sử dụng chi tiết, vui lòng tham khảo tài liệu chính thức bằng tiếng Trung. Tài liệu này chỉ giải thích cách cấu hình và sử dụng Casbin trong webman.

https://casbin.org/docs/zh-CN/overview

## Cấu trúc thư mục

```
.
├── config                        Thư mục cấu hình
│   ├── casbin-restful-model.conf Tệp cấu hình mô hình quyền
│   ├── casbin.php                Cấu hình Casbin
......
├── database                      Tệp cơ sở dữ liệu
│   ├── migrations                Tệp di cư
│   │   └── 20210218074218_create_rule_table.php
......
```

## Tệp di cư cơ sở dữ liệu

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
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'Bảng quy tắc']);

        // Thêm các trường dữ liệu
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => 'ID khóa chính'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'Loại quy tắc'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // Thực thi tạo
        $table->create();
    }
}
```

## Cấu hình Casbin

Để xem cú pháp cấu hình mô hình quy tắc quyền, tham khảo: https://casbin.org/docs/zh-CN/syntax-for-models

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // Tệp cấu hình mô hình quy tắc quyền
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // Có thể cấu hình nhiều mô hình quyền
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // Tệp cấu hình mô hình quy tắc quyền
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\RBACRule::class,
        ],
    ],
];
```

### Bộ chuyển đổi

Gói Composer hiện tại được thiết lập tương thích với các phương thức model của think-orm. Đối với các ORM khác, tham khảo vendor/teamones/src/adapters/DatabaseAdapter.php

Sau đó sửa đổi cấu hình:

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // Tệp cấu hình mô hình quy tắc quyền
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // Cấu hình loại ở chế độ bộ chuyển đổi tại đây
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## Hướng dẫn sử dụng

### Nhập

```php
# Nhập
use teamones\casbin\Enforcer;
```

### Hai cách sử dụng

```php
# 1. Sử dụng cấu hình mặc định
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. Sử dụng cấu hình rbac tùy chỉnh
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### Giới thiệu API thông dụng

Để biết thêm cách sử dụng API, vui lòng tham khảo tài liệu chính thức:

- Management API: https://casbin.org/docs/zh-CN/management-api
- RBAC API: https://casbin.org/docs/zh-CN/rbac-api

```php
# Thêm quyền cho người dùng

Enforcer::addPermissionForUser('user1', '/user', 'read');

# Xóa quyền của người dùng

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# Lấy tất cả quyền của người dùng

Enforcer::getPermissionsForUser('user1');

# Thêm vai trò cho người dùng

Enforcer::addRoleForUser('user1', 'role1');

# Thêm quyền cho vai trò

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# Lấy tất cả vai trò

Enforcer::getAllRoles();

# Lấy tất cả vai trò của người dùng

Enforcer::getRolesForUser('user1');

# Lấy người dùng theo vai trò

Enforcer::getUsersForRole('role1');

# Kiểm tra người dùng có thuộc vai trò hay không

Enforcer::hasRoleForUser('user1', 'role1');

# Xóa vai trò của người dùng

Enforcer::deleteRoleForUser('user1', 'role1');

# Xóa tất cả vai trò của người dùng

Enforcer::deleteRolesForUser('user1');

# Xóa vai trò

Enforcer::deleteRole('role1');

# Xóa quyền

Enforcer::deletePermission('/user', 'read');

# Xóa tất cả quyền của người dùng hoặc vai trò

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# Kiểm tra quyền, trả về true hoặc false

Enforcer::enforce("user1", "/user", "edit");
```
