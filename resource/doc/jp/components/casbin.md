# Casbin

## 説明

Casbinは、強力で効率的なオープンソースのアクセス制御フレームワークです。その権限管理メカニズムは複数のアクセス制御モデルをサポートしています。

## プロジェクトアドレス

https://github.com/teamones-open/casbin

## インストール

```php
composer require teamones/casbin
```

## Casbin公式ウェブサイト

詳細な使用方法は公式中国語ドキュメントを参照してください。このドキュメントではwebmanでの設定と使用方法のみを説明します。

https://casbin.org/docs/zh-CN/overview

## ディレクトリ構造

```
.
├── config                        設定ディレクトリ
│   ├── casbin-restful-model.conf 権限モデルの設定ファイル
│   ├── casbin.php                Casbin設定
......
├── database                      データベースファイル
│   ├── migrations                マイグレーションファイル
│   │   └── 20210218074218_create_rule_table.php
......
```

## データベースマイグレーションファイル

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
        $table = $this->table('rule', ['id' => false, 'primary_key' => ['id'], 'engine' => 'InnoDB', 'collation' => 'utf8mb4_general_ci', 'comment' => 'ルールテーブル']);

        // データフィールドを追加
        $table->addColumn('id', 'integer', ['identity' => true, 'signed' => false, 'limit' => 11, 'comment' => '主キーID'])
            ->addColumn('ptype', 'char', ['default' => '', 'limit' => 8, 'comment' => 'ルールタイプ'])
            ->addColumn('v0', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v1', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v2', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v3', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v4', 'string', ['default' => '', 'limit' => 128])
            ->addColumn('v5', 'string', ['default' => '', 'limit' => 128]);

        // 作成を実行
        $table->create();
    }
}
```

## Casbin設定

権限ルールモデルの設定構文については: https://casbin.org/docs/zh-CN/syntax-for-models を参照してください。

```php
<?php

return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // 権限ルールモデル設定ファイル
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\Rule::class,
        ],
    ],
    // 複数の権限モデルを設定可能
    'rbac' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-rbac-model.conf', // 権限ルールモデル設定ファイル
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'model', // model or adapter
            'class' => \app\model\RBACRule::class,
        ],
    ],
];
```

### アダプター

現在のComposerパッケージはthink-ormのmodelメソッドに適応しています。他のORMについてはvendor/teamones/src/adapters/DatabaseAdapter.phpを参照してください。

その後、設定を変更してください。

```php
return [
    'default' => [
        'model' => [
            'config_type' => 'file',
            'config_file_path' => config_path() . '/casbin-restful-model.conf', // 権限ルールモデル設定ファイル
            'config_text' => '',
        ],
        'adapter' => [
            'type' => 'adapter', // ここでタイプをアダプターモードに設定
            'class' => \app\adapter\DatabaseAdapter::class,
        ],
    ],
];
```

## 使用方法

### インポート

```php
# インポート
use teamones\casbin\Enforcer;
```

### 2つの使い方

```php
# 1. デフォルト設定を使用
Enforcer::addPermissionForUser('user1', '/user', 'read');

# 2. カスタムrbac設定を使用
Enforcer::instance('rbac')->addPermissionForUser('user1', '/user', 'read');
```

### よく使うAPIの紹介

より多くのAPIの使い方は公式ドキュメントを参照してください。

- 管理API: https://casbin.org/docs/zh-CN/management-api
- RBAC API: https://casbin.org/docs/zh-CN/rbac-api

```php
# ユーザーに権限を追加

Enforcer::addPermissionForUser('user1', '/user', 'read');

# ユーザーの権限を削除

Enforcer::deletePermissionForUser('user1', '/user', 'read');

# ユーザーの全ての権限を取得

Enforcer::getPermissionsForUser('user1');

# ユーザーにロールを追加

Enforcer::addRoleForUser('user1', 'role1');

# ロールに権限を追加

Enforcer::addPermissionForUser('role1', '/user', 'edit');

# 全てのロールを取得

Enforcer::getAllRoles();

# ユーザーの全てのロールを取得

Enforcer::getRolesForUser('user1');

# ロールからユーザーを取得

Enforcer::getUsersForRole('role1');

# ユーザーがロールに属するか確認

Enforcer::hasRoleForUser('user1', 'role1');

# ユーザーのロールを削除

Enforcer::deleteRoleForUser('user1', 'role1');

# ユーザーの全てのロールを削除

Enforcer::deleteRolesForUser('user1');

# ロールを削除

Enforcer::deleteRole('role1');

# 権限を削除

Enforcer::deletePermission('/user', 'read');

# ユーザーまたはロールの全ての権限を削除

Enforcer::deletePermissionsForUser('user1');
Enforcer::deletePermissionsForUser('role1');

# 権限をチェック、trueまたはfalseを返す

Enforcer::enforce("user1", "/user", "edit");
```
