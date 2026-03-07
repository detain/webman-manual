# Bağımlılık Otomatik Enjeksiyonu

webman'de bağımlılık otomatik enjeksiyonu isteğe bağlı bir özelliktir ve varsayılan olarak kapalıdır. Bağımlılık otomatik enjeksiyonuna ihtiyacınız varsa [php-di](https://php-di.org/doc/getting-started.html) kullanılması önerilir. Aşağıda webman ile `php-di` kullanımı anlatılmaktadır.

## Kurulum

```
composer require php-di/php-di:^7.0
```

`config/container.php` yapılandırmasını düzenleyin. Son içerik şöyle olmalıdır:

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> `config/container.php` dosyası nihai olarak PSR-11 spesifikasyonuna uygun bir konteyner örneği döndürmelidir. `php-di` kullanmak istemiyorsanız burada PSR-11 uyumlu başka bir konteyner örneği oluşturup döndürebilirsiniz. Varsayılan yapılandırma yalnızca webman'in temel konteyner işlevselliğini sağlar.

## Konstruktör Enjeksiyonu

`app/service/Mailer.php` dosyasını oluşturun (yoksa dizini oluşturun) ve içeriğini şöyle yapın:

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // E-posta gönderme kodu atlanmıştır
    }
}
```

`app/controller/UserController.php` içeriği:

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
        $this->mailer->mail('hello@webman.com', 'Merhaba ve hoş geldiniz!');
        return response('ok');
    }
}
```

Normalde `app\controller\UserController` örneğini oluşturmak için şu kod gerekir:

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

`php-di` kullanıldığında geliştiricilerin controller içindeki `Mailer`'ı manuel olarak örneklemesine gerek yoktur — webman bunu otomatik yapar. `Mailer` örneği oluşturulurken başka sınıf bağımlılıkları varsa, webman onları da otomatik örnekleyip enjekte eder. Geliştiriciden herhangi bir başlatma çalışması gerekmez.

> **Not**
> Bağımlılık otomatik enjeksiyonunu yalnızca framework veya `php-di` tarafından oluşturulan örnekler destekler. Manuel `new` ile oluşturulan örnekler desteklemez. Enjeksiyon için `new` yerine `support\Container` arayüzünü kullanın, örneğin:

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// new ile oluşturulan örnekler bağımlılık enjeksiyonunu desteklemez
$user_service = new UserService;
// new ile oluşturulan örnekler bağımlılık enjeksiyonunu desteklemez
$log_service = new LogService($path, $name);

// Container ile oluşturulan örnekler bağımlılık enjeksiyonunu destekler
$user_service = Container::get(UserService::class);
// Container ile oluşturulan örnekler bağımlılık enjeksiyonunu destekler
$log_service = Container::make(LogService::class, [$path, $name]);
```

## Öznitelik Enjeksiyonu (Attributes)

Konstruktör bağımlılık enjeksiyonuna ek olarak öznitelik enjeksiyonu da kullanılabilir. Önceki örneğe devam ederek `app\controller\UserController`'ı şöyle değiştirin:

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
        $this->mailer->mail('hello@webman.com', 'Merhaba ve hoş geldiniz!');
        return response('ok');
    }
}
```

Bu örnek enjeksiyon için `#[Inject]` özniteliğini kullanır ve nesne türüne göre örneği üye değişkene otomatik enjekte eder. Konstruktör enjeksiyonu ile aynı etkiye sahiptir ancak kod daha özlüdür.

> **Not**
> webman 1.4.6 sürümünden önce controller parametre enjeksiyonunu desteklemez. Örneğin aşağıdaki kod webman<=1.4.6 iken desteklenmez:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // 1.4.6 öncesi controller parametre enjeksiyonu desteklenmez
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'Merhaba ve hoş geldiniz!');
        return response('ok');
    }
}
```

## Özel Konstruktör Enjeksiyonu

Bazen konstruktöre geçirilen parametreler sınıf örnekleri değil, dize, sayı, dizi vb. veriler olabilir. Örneğin Mailer konstruktörü SMTP sunucu IP'si ve portuna ihtiyaç duyabilir:

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
        // E-posta gönderme kodu atlanmıştır
    }
}
```

Bu durumda önceki konstruktör otomatik enjeksiyonu doğrudan kullanılamaz çünkü `php-di` `$smtp_host` ve `$smtp_port` değerlerini belirleyemez. Bu durumda özel enjeksiyon kullanılabilir.

`config/dependence.php` dosyasına (yoksa oluşturun) şu kodu ekleyin:

```php
return [
    // ... Diğer yapılandırmalar atlanmıştır

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

Bağımlılık enjeksiyonu `app\service\Mailer` örneğine ihtiyaç duyduğunda bu yapılandırmada oluşturulan örneği otomatik kullanacaktır.

`config/dependence.php`'nin `Mailer` sınıfını örneklemek için `new` kullandığını unutmayın. Bu örnekte sorun yoktur ancak `Mailer` sınıfı başka sınıflara bağımlıysa veya dahili olarak öznitelik enjeksiyonu kullanıyorsa, `new` ile başlatma bağımlılık otomatik enjeksiyonunu yapmayacaktır. Çözüm özel arayüz enjeksiyonu kullanmak ve sınıfları `Container::get(sınıf adı)` veya `Container::make(sınıf adı, [konstruktör parametreleri])` ile başlatmaktır.

## Özel Arayüz Enjeksiyonu

Gerçek projelerde somut sınıflar yerine arayüzlere karşı programlama tercih edilir. Örneğin `app\controller\UserController` `app\service\Mailer` yerine `app\service\MailerInterface`'e bağımlı olmalıdır.

`MailerInterface` arayüzünü tanımlayın:

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

`MailerInterface` uygulamasını tanımlayın:

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
        // E-posta gönderme kodu atlanmıştır
    }
}
```

Somut uygulama yerine `MailerInterface` arayüzünü kullanın:

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
        $this->mailer->mail('hello@webman.com', 'Merhaba ve hoş geldiniz!');
        return response('ok');
    }
}
```

`config/dependence.php`'de `MailerInterface` uygulamasını tanımlayın:

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

Uygulama `MailerInterface` arayüzüne ihtiyaç duyduğunda otomatik olarak `Mailer` uygulaması kullanılacaktır.

> Arayüzlere karşı programlamanın avantajı, bir bileşen değiştirilmesi gerektiğinde iş kodunun değiştirilmesi gerekmez — yalnızca `config/dependence.php`'deki somut uygulama değişir. Birim testleri için de çok faydalıdır.

## Diğer Özel Enjeksiyonlar

`config/dependence.php` sınıf bağımlılıkları tanımlamanın yanı sıra dize, sayı, dizi vb. değerler de tanımlayabilir.

Örneğin `config/dependence.php` şöyle tanımlanmışsa:

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

`#[Inject]` ile `smtp_host` ve `smtp_port`'u sınıf özelliklerine enjekte edebilirsiniz:

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
        // E-posta gönderme kodu atlanmıştır
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // 192.168.1.11:25 çıktısı verir
    }
}
```

# Tembel Yükleme (Lazy Loading)

> Tembel yükleme, nesnelerin oluşturulmasını veya başlatılmasını gerçekten gerekene kadar erteleyen bir tasarım kalıbıdır.

Bu özellik ek bir bağımlılık gerektirir. Aşağıdaki paket `ocramius/proxy-manager`'ın bir fork'udur; orijinal depo PHP 8'i desteklemez.

```
composer require friendsofphp/proxy-manager-lts
```

Kullanım:

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
        echo "MyClass örneklendi\n";
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
        echo "Vekil sınıf adı: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

Çıktı:

```
Vekil sınıf adı: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass örneklendi
name: Lazy Loaded Object
```

Bu örnek `#[Injectable]` özniteliğiyle bildirilen sınıf enjekte edildiğinde önce bu sınıfın vekil sınıfının oluşturulduğunu, gerçek sınıfın yalnızca herhangi bir metodu çağrıldığında örneklendiğini gösterir.

# Döngüsel Bağımlılıklar

Döngüsel bağımlılıklar birden fazla sınıfın birbirine bağımlı olup kapalı bir bağımlılık döngüsü oluşturduğunda ortaya çıkar.

- Doğrudan döngüsel bağımlılık
  - Modül A modül B'ye, modül B modül A'ya bağımlı
  - Döngü oluşturur: A → B → A

- Dolaylı döngüsel bağımlılık
  - Birden fazla modülün bağımlılık döngüsünde yer alması
  - Örn: A → B → C → A

Öznitelik enjeksiyonu kullanıldığında `php-di` döngüsel bağımlılıkları otomatik tespit eder ve istisna fırlatır. Gerekirse şu yaklaşımı kullanın:

```php
class userController
{

    // Bu kodu kaldırın
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## Daha Fazla Bilgi

Lütfen [php-di belgelerine](https://php-di.org/doc/getting-started.html) bakın.
