# 依存の自動インジェクション

webmanにおける依存の自動インジェクションはオプション機能であり、デフォルトでは無効になっています。依存の自動インジェクションが必要な場合は、[php-di](https://php-di.org/doc/getting-started.html)の使用を推奨します。以下は`php-di`とwebmanの連携方法です。

## インストール

```
composer require php-di/php-di:^7.0
```

`config/container.php`を変更します。最終的な内容は以下のとおりです：

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> `config/container.php`は最終的にPSR-11規格に準拠したコンテナインスタンスを返す必要があります。`php-di`を使わない場合は、ここで他のPSR-11準拠のコンテナインスタンスを作成して返すことができます。デフォルト設定はwebmanの基本的なコンテナ機能のみを提供します。

## コンストラクタインジェクション

`app/service/Mailer.php`を新規作成します（ディレクトリが存在しない場合は作成してください）。内容は以下のとおりです：

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // メール送信コードは省略
    }
}
```

`app/controller/UserController.php`の内容は以下のとおりです：

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{

    public function __construct(private Mailer $mailer)
    {
    }

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

通常、`app\controller\UserController`のインスタンス化には以下のコードが必要です：

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

`php-di`を使用すると、開発者はコントローラー内の`Mailer`を手動でインスタンス化する必要がなく、webmanが自動的に処理します。`Mailer`のインスタンス化過程で他のクラスへの依存がある場合も、webmanが自動的にインスタンス化して注入します。開発者は初期化作業を行う必要がありません。

> **注意**
> 依存の自動インジェクションが完了するのは、フレームワークまたは`php-di`が作成したインスタンスに限ります。`new`で手動作成したインスタンスでは依存の自動インジェクションはできません。インジェクションが必要な場合は、`new`文の代わりに`support\Container`インターフェースを使用してください。例：

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// newキーワードで作成したインスタンスでは依存インジェクション不可
$user_service = new UserService;
// newキーワードで作成したインスタンスでは依存インジェクション不可
$log_service = new LogService($path, $name);

// Containerで作成したインスタンスでは依存インジェクション可能
$user_service = Container::get(UserService::class);
// Containerで作成したインスタンスでは依存インジェクション可能
$log_service = Container::make(LogService::class, [$path, $name]);
```

## 属性インジェクション

コンストラクタ依存の自動インジェクションに加え、属性インジェクションも使用できます。上記の例を続けて、`app\controller\UserController`を以下のように変更します：

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private Mailer $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

この例では`#[Inject]`属性によるインジェクションを使用し、オブジェクト型に基づいてインスタンスがメンバー変数に自動注入されます。コンストラクタインジェクションと同等の効果で、コードはより簡潔です。

> **注意**
> webman 1.4.6以前ではコントローラーパラメータのインジェクションをサポートしていません。例えば以下のコードはwebman<=1.4.6ではサポートされません：

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // 1.4.6以前はコントローラーパラメータインジェクション非対応
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

## カスタムコンストラクタインジェクション

コンストラクタに渡す引数がクラスのインスタンスではなく、文字列・数値・配列などの非オブジェクトデータの場合があります。例えば、MailerのコンストラクタにSMTPサーバーのIPとポートを渡す必要がある場合：

```php
<?php
namespace app\service;

class Mailer
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // メール送信コードは省略
    }
}
```

このような場合は、前述のコンストラクタ自動インジェクションを直接使えません。`php-di`が`$smtp_host`と`$smtp_port`の値を特定できないためです。この場合はカスタムインジェクションを試してください。

`config/dependence.php`（ファイルがなければ作成）に以下のコードを追加します：

```php
return [
    // ... 他の設定は省略

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

これにより、依存インジェクションで`app\service\Mailer`インスタンスが必要になったときに、この設定で作成した`app\service\Mailer`インスタンスが自動的に使われます。

`config/dependence.php`では`new`で`Mailer`クラスをインスタンス化しています。この例では問題ありませんが、`Mailer`クラスが他のクラスに依存していたり、クラス内で属性インジェクションを使っていたりする場合は、`new`での初期化では依存の自動インジェクションは行われません。解決策は、カスタムインターフェースインジェクションを使い、`Container::get(クラス名)`または`Container::make(クラス名, [コンストラクタ引数])`でクラスを初期化することです。

## カスタムインターフェースインジェクション

実際のプロジェクトでは、具象クラスではなくインターフェースに基づいてプログラミングする方が望ましいです。例えば、`app\controller\UserController`では`app\service\Mailer`ではなく`app\service\MailerInterface`を参照すべきです。

`MailerInterface`インターフェースを定義します：

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

`MailerInterface`の実装を定義します：

```php
<?php
namespace app\service;

class Mailer implements MailerInterface
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // メール送信コードは省略
    }
}
```

具象実装ではなく`MailerInterface`インターフェースを参照します：

```php
<?php
namespace app\controller;

use support\Request;
use app\service\MailerInterface;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private MailerInterface $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

`config/dependence.php`で`MailerInterface`の実装を定義します：

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

これにより、ビジネスロジックで`MailerInterface`が必要な場合、自動的に`Mailer`の実装が使われます。

> インターフェース指向プログラミングの利点は、コンポーネントを差し替える際にビジネスコードを変更せず、`config/dependence.php`の具象実装だけを変更すればよい点です。単体テストにも非常に有用です。

## その他のカスタムインジェクション

`config/dependence.php`ではクラス依存だけでなく、文字列・数値・配列などの他の値も定義できます。

例えば、`config/dependence.php`を以下のように定義した場合：

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

`#[Inject]`で`smtp_host`と`smtp_port`をクラスのプロパティに注入できます：

```php
<?php
namespace app\service;

use DI\Attribute\Inject;

class Mailer
{
    #[Inject("smtp_host")]
    private $smtpHost;

    #[Inject("smtp_port")]
    private $smtpPort;

    public function mail($email, $content)
    {
        // メール送信コードは省略
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // 192.168.1.11:25 と出力
    }
}
```

# 遅延ロード（Lazy Loading）

> 遅延ロードは、オブジェクトの作成や初期化を実際に必要になるまで先送りする設計パターンです。

この機能を使うには追加の依存パッケージが必要です。以下は`ocramius/proxy-manager`のフォークで、元のリポジトリはPHP 8をサポートしていません。

```
composer require friendsofphp/proxy-manager-lts
```

使用例：

```php
<?php

use DI\Attribute\Injectable;
use DI\Attribute\Inject;

#[Injectable(lazy: true)]
class MyClass
{
    private string $name;

    public function __construct()
    {
        echo "MyClass インスタンス化\n";
        $this->name = "Lazy Loaded Object";
    }

    public function getName(): string
    {
        return $this->name;
    }
}

class Controller
{
    #[Inject]
    public MyClass $myClass;

    public function getClass()
    {
        echo "プロキシクラス名: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

出力：

```
プロキシクラス名: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass インスタンス化
name: Lazy Loaded Object
```

この例から分かるように、`#[Injectable]`属性を付けたクラスは注入されるとき、まずそのクラスのプロキシクラスが作成され、いずれかのメソッドが呼ばれた時点で初めてインスタンス化されます。

# 循環依存

循環依存とは、複数のクラスが互いに依存し、閉じた依存関係を形成する状態です。

- 直接的な循環依存
  - モジュールAがモジュールBに依存し、モジュールBがモジュールAに依存
  - A → B → A の依存ループを形成

- 間接的な循環依存
  - 複数モジュールが関与する依存ループ
  - 例：A → B → C → A

属性インジェクションを使う場合、`php-di`は循環依存を自動検出し、例外をスローします。必要な場合は、以下のように記述してください：

```php
class userController
{

    // 以下の行を削除
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## 関連ドキュメント

[php-diマニュアル](https://php-di.org/doc/getting-started.html)を参照してください。
