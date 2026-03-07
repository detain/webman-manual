# 依賴自動注入
在 webman 裡，依賴自動注入是選用功能，此功能預設關閉。若需依賴自動注入，建議使用 [php-di](https://php-di.org/doc/getting-started.html)，以下說明如何將 `php-di` 與 webman 搭配使用。

## 安裝
```
composer require php-di/php-di:^7.0
```

修改 `config/container.php` 設定，其最終內容如下：

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> `config/container.php` 最終必須回傳符合 `PSR-11` 規範的容器實例。若不想使用 `php-di`，可在此建立並回傳其他符合 `PSR-11` 規範的容器實例。預設設定僅提供 webman 基礎容器功能。

## 建構函式注入
新增 `app/service/Mailer.php`（若目錄不存在請自行建立），內容如下：

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // 發送郵件程式碼略
    }
}
```

`app/controller/UserController.php` 內容如下：

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

一般情況下，需以下程式碼才能完成 `app\controller\UserController` 的實例化：

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

使用 `php-di` 後，開發者不需手動實例化控制器中的 `Mailer`，webman 會自動處理。若實例化 `Mailer` 時有其它類別依賴，webman 也會自動實例化並注入，開發者無需做任何初始化。

> **注意**
> 必須是由框架或 `php-di` 建立的實例才能完成依賴自動注入，使用 `new` 手動建立的實例無法完成依賴自動注入。若需注入，請以 `support\Container` 介面取代 `new` 陳述式，例如：

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// 使用 new 關鍵字建立的實例無法進行依賴注入
$user_service = new UserService;
// 使用 new 關鍵字建立的實例無法進行依賴注入
$log_service = new LogService($path, $name);

// 使用 Container 建立的實例可進行依賴注入
$user_service = Container::get(UserService::class);
// 使用 Container 建立的實例可進行依賴注入
$log_service = Container::make(LogService::class, [$path, $name]);
```

## 屬性注入
除了建構函式依賴自動注入，也可使用屬性注入。延續上例，將 `app\controller\UserController` 改成如下：

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

此例使用 `#[Inject]` 屬性注入，依物件型別自動將實例注入成員變數。效果與建構函式注入相同，但程式碼更精簡。

> **注意**
> webman 在 1.4.6 版之前不支援控制器參數注入，例如以下程式碼在 webman<=1.4.6 時不支援：

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // 1.4.6 版之前不支援控制器參數注入
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

## 自訂建構函式注入
有時建構函式參數可能不是類別實例，而是字串、數字、陣列等非物件資料。例如 Mailer 建構函式需傳入 SMTP 伺服器 IP 與埠號：

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
        // 發送郵件程式碼略
    }
}
```

這種情況無法直接使用前述建構函式自動注入，因為 `php-di` 無法得知 `$smtp_host`、`$smtp_port` 的值。此時可使用自訂注入。

在 `config/dependence.php`（若檔案不存在請自行建立）加入下列程式碼：

```php
return [
    // ... 此處略過其它設定

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

當依賴注入需要取得 `app\service\Mailer` 實例時，會自動使用此設定中建立的 `app\service\Mailer` 實例。

需注意，`config/dependence.php` 中用 `new` 實例化 `Mailer` 類別，在本範例沒問題。但若 `Mailer` 依賴其它類別，或內部使用屬性注入，以 `new` 初始化將不會進行依賴自動注入。解決方式為自訂介面注入，透過 `Container::get(類別名稱)` 或 `Container::make(類別名稱, [建構函式參數])` 初始化類別。

## 自訂介面注入
在實際專案中，較建議以介面導向程式設計，而非具體類別。例如 `app\controller\UserController` 應依賴 `app\service\MailerInterface`，而非 `app\service\Mailer`。

定義 `MailerInterface` 介面：

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

定義 `MailerInterface` 的實作：

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
        // 發送郵件程式碼略
    }
}
```

使用 `MailerInterface` 介面，而非具體實作：

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

在 `config/dependence.php` 中定義 `MailerInterface` 的實作：

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

當業務需使用 `MailerInterface` 介面時，會自動使用 `Mailer` 實作。

> 以介面導向程式設計的好處是，更換元件時不需修改業務程式碼，只需修改 `config/dependence.php` 中的具體實作。對單元測試也很有幫助。

## 其它自訂注入
`config/dependence.php` 除可定義類別依賴外，也可定義字串、數字、陣列等其它值。

例如 `config/dependence.php` 定義如下：

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

此時可透過 `#[Inject]` 將 `smtp_host`、`smtp_port` 注入類別屬性：

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
        // 發送郵件程式碼略
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // 將輸出 192.168.1.11:25
    }
}
```

# 延遲載入
> 延遲載入是一種設計模式，可將物件的建立或初始化延遲到實際需要時再進行。

使用此功能需額外安裝相依套件，下列為 `ocramius/proxy-manager` 的分支，原套件不支援 PHP 8：

```
composer require friendsofphp/proxy-manager-lts
```

使用方式：

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
        echo "MyClass 實例化\n";
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
        echo "代理物件類別名稱: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

輸出：

```
代理物件類別名稱: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass 實例化
name: Lazy Loaded Object
```

以上範例說明，宣告 `#[Injectable]` 屬性的類別在被注入時，會先建立該類別的代理類別，只有在任意方法被呼叫後才會實例化。

# 迴圈依賴
迴圈依賴是指多個類別相互依賴，形成封閉的依賴關係。

- 直接迴圈依賴
  - 模組 A 依賴模組 B，模組 B 又依賴模組 A
  - 形成 A → B → A 的依賴迴圈

- 間接迴圈依賴
  - 涉及多個模組形成的依賴環
  - 例如 A → B → C → A

使用屬性注入時，`php-di` 會自動偵測迴圈依賴並拋出例外。若需使用，請改為以下寫法：

```php
class userController
{

    // 移除以下程式碼
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## 更多內容
請參考 [php-di 手冊](https://php-di.org/doc/getting-started.html)
