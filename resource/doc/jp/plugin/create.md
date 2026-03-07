# 基本プラグインの生成および公開フロー

## 原理
1. クロスドメインプラグインを例にすると、プラグインは3つに分かれます。クロスドメインミドルウェアのプログラムファイル、ミドルウェア設定ファイル `middleware.php`、そしてコマンドで自動生成される `Install.php` です。
2. コマンドを使ってこの3つのファイルをパッケージ化し、Composer に公開します。
3. ユーザーが Composer でクロスドメインプラグインをインストールすると、プラグイン内の `Install.php` がミドルウェアのプログラムファイルと設定ファイルを `{メインプロジェクト}/config/plugin` にコピーし、webman が読み込めるようにします。これによりクロスドメインミドルウェアの自動設定が有効になります。
4. ユーザーが Composer でプラグインを削除すると、`Install.php` が対応するミドルウェアファイルと設定ファイルを削除し、プラグインの自動アンインストールを行います。

## 規約
1. プラグイン名は `ベンダー` と `プラグイン名` の2つで構成されます。例: `webman/push`。Composer パッケージ名に対応します。
2. プラグインの設定ファイルは `config/plugin/ベンダー/プラグイン名/` に統一して配置します（console コマンドが設定ディレクトリを自動作成）。設定が不要な場合は、自動作成された設定ディレクトリを削除してください。
3. プラグイン設定ディレクトリでサポートされるのは app.php（メイン設定）、bootstrap.php（プロセス起動）、route.php（ルート）、middleware.php（ミドルウェア）、process.php（カスタムプロセス）、database.php（DB）、redis.php（Redis）、thinkorm.php（thinkorm）のみです。これらは webman に自動認識されます。
4. 設定の取得例: `config('plugin.ベンダー.プラグイン名.設定ファイル.項目');`。例: `config('plugin.webman.push.app.app_key')`。
5. 独自の DB 設定を持つ場合: `illuminate/database` は `Db::connection('plugin.ベンダー.プラグイン名.接続名')`、`thinkorm` は `Db::connect('plugin.ベンダー.プラグイン名.接続名')` でアクセスします。
6. `app/` に業務ファイルを置く場合は、メインプロジェクトや他プラグインと競合しないようにしてください。
7. 可能な限りメインプロジェクトへファイルやディレクトリをコピーしないでください。例: クロスドメインプラグインの場合、設定ファイル以外は `vendor/webman/cros/src` に置き、メインプロジェクトへコピーする必要はありません。
8. プラグインの名前空間は PascalCase を推奨します。例: `Webman/Console`。

## 例

**`webman/console` のインストール**

`composer require webman/console`

### プラグインの作成

作成するプラグイン名を `foo/admin` とします（Composer で公開するプロジェクト名でもあり、小文字にする必要があります）。次を実行します:

`php webman plugin:create --name=foo/admin`

作成後、`vendor/foo/admin`（プラグインファイル用）と `config/plugin/foo/admin`（プラグイン設定用）が生成されます。

> 注意
> `config/plugin/foo/admin` でサポートされるのは app.php、bootstrap.php、route.php、middleware.php、process.php、database.php、redis.php、thinkorm.php です。形式は webman と同じで、自動的にマージされます。
> アクセス時は `plugin` をプレフィックスにします。例: `config('plugin.foo.admin.app')`


### プラグインのエクスポート

開発完了後、次を実行してエクスポートします:

`php webman plugin:export --name=foo/admin`

エクスポート

> 説明
> エクスポートすると `config/plugin/foo/admin` が `vendor/foo/admin/src` にコピーされ、`Install.php` が自動生成されます。`Install.php` はインストール／アンインストール時に実行されます。
> インストール時: `vendor/foo/admin/src` の設定をプロジェクトの `config/plugin` へコピーします。
> アンインストール時: プロジェクトの `config/plugin` 配下の該当設定ファイルを削除します。
> カスタム処理は `Install.php` を編集して追加できます。

### プラグインの公開
* [GitHub](https://github.com) と [Packagist](https://packagist.org) のアカウントがあるとします。
* [GitHub](https://github.com) で `admin` リポジトリを作成し、コードをプッシュします。例: `https://github.com/あなたのユーザー名/admin`。
* `https://github.com/あなたのユーザー名/admin/releases/new` で release を作成します。例: `v1.0.0`。
* [Packagist](https://packagist.org) の `Submit` をクリックし、`https://github.com/あなたのユーザー名/admin` を登録すると公開完了です。

> **ヒント**
> Packagist で名前の競合が出る場合は、ベンダー名を変更してください。例: `foo/admin` を `myfoo/admin` に変更。

更新時は GitHub にプッシュし、`https://github.com/あなたのユーザー名/admin/releases/new` で新しい release を作成してから、`https://packagist.org/packages/foo/admin` の `Update` をクリックしてバージョンを更新します。

## プラグインにコマンドを追加する
プラグインにカスタムコマンドを追加して補助機能を提供することがあります。例: `webman/redis-queue` をインストールすると、`redis-queue:consumer` コマンドが追加され、ユーザーが `php webman redis-queue:consumer send-mail` を実行すると `SendMail.php` のコンシューマークラスが生成され、開発が簡単になります。

`foo/admin` に `foo-admin:add` コマンドを追加する場合の手順です。

### コマンドの作成

**`vendor/foo/admin/src/FooAdminAddCommand.php` を作成**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = 'コマンドの説明';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **注意**
> プラグイン間のコマンド競合を避けるため、コマンド形式は `ベンダー-プラグイン:コマンド` を推奨します。`foo/admin` の全コマンドは `foo-admin:` をプレフィックスにします。例: `foo-admin:add`。

### 設定の追加
**`config/plugin/foo/admin/command.php` を作成**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // 必要に応じて追加...
];
```

> **ヒント**
> `command.php` はプラグインのカスタムコマンドを登録します。配列の各要素はコマンドクラスです。`webman/console` がこれらのコマンドを自動で読み込みます。詳細は [コンソール](console.md) を参照してください。

### エクスポートの実行
`php webman plugin:export --name=foo/admin` を実行してエクスポートし、Packagist に公開します。`foo/admin` インストール後、`foo-admin:add` コマンドが使えます。`php webman foo-admin:add jerry` を実行すると `Admin add jerry` と表示されます。
