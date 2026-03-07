# 의존성 자동 주입

webman에서 의존성 자동 주입은 선택 기능이며 기본적으로 비활성화되어 있습니다. 의존성 자동 주입이 필요하다면 [php-di](https://php-di.org/doc/getting-started.html) 사용을 권장합니다. 아래는 webman과 `php-di`를 함께 사용하는 방법입니다.

## 설치

```
composer require php-di/php-di:^7.0
```

`config/container.php` 설정을 수정합니다. 최종 내용은 다음과 같습니다:

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> `config/container.php`는 최종적으로 PSR-11 규격에 맞는 컨테이너 인스턴스를 반환해야 합니다. `php-di`를 사용하지 않으려면 여기서 다른 PSR-11 준수 컨테이너 인스턴스를 생성하여 반환할 수 있습니다. 기본 설정은 webman의 기본 컨테이너 기능만 제공합니다.

## 생성자 주입

`app/service/Mailer.php`를 새로 만듭니다(디렉토리가 없으면 생성하세요). 내용은 다음과 같습니다:

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // 이메일 전송 코드 생략
    }
}
```

`app/controller/UserController.php` 내용은 다음과 같습니다:

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

일반적으로 `app\controller\UserController` 인스턴스화에는 다음 코드가 필요합니다:

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

`php-di`를 사용하면 개발자가 컨트롤러 내 `Mailer`를 수동으로 인스턴스화할 필요가 없으며, webman이 자동으로 처리합니다. `Mailer` 인스턴스화 과정에 다른 클래스 의존성이 있어도 webman이 자동으로 인스턴스화하고 주입합니다. 개발자는 초기화 작업이 필요 없습니다.

> **참고**
> 의존성 자동 주입을 완료하려면 프레임워크 또는 `php-di`가 생성한 인스턴스여야 합니다. 수동으로 `new`로 생성한 인스턴스는 의존성 자동 주입을 할 수 없습니다. 주입이 필요하면 `new` 문 대신 `support\Container` 인터페이스를 사용하세요. 예:

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// new 키워드로 생성한 인스턴스는 의존성 주입 불가
$user_service = new UserService;
// new 키워드로 생성한 인스턴스는 의존성 주입 불가
$log_service = new LogService($path, $name);

// Container로 생성한 인스턴스는 의존성 주입 가능
$user_service = Container::get(UserService::class);
// Container로 생성한 인스턴스는 의존성 주입 가능
$log_service = Container::make(LogService::class, [$path, $name]);
```

## 속성 주입

생성자 의존성 자동 주입 외에 속성 주입도 사용할 수 있습니다. 위 예시를 이어서 `app\controller\UserController`를 다음과 같이 변경합니다:

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

이 예시는 `#[Inject]` 속성으로 주입하며, 객체 타입에 따라 인스턴스가 멤버 변수에 자동으로 주입됩니다. 생성자 주입과 동일한 효과이지만 코드가 더 간결합니다.

> **참고**
> webman 1.4.6 이전 버전에서는 컨트롤러 인자 주입을 지원하지 않습니다. 예를 들어 다음 코드는 webman<=1.4.6에서 지원되지 않습니다:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // 1.4.6 버전 이전은 컨트롤러 인자 주입 미지원
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

## 사용자 정의 생성자 주입

생성자에 전달되는 인자가 클래스 인스턴스가 아닌 문자열, 숫자, 배열 등의 비객체 데이터인 경우가 있습니다. 예를 들어 Mailer 생성자에 SMTP 서버 IP와 포트를 전달해야 하는 경우:

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
        // 이메일 전송 코드 생략
    }
}
```

이런 경우 앞서 소개한 생성자 자동 주입을 직접 사용할 수 없습니다. `php-di`가 `$smtp_host`와 `$smtp_port`의 값을 알 수 없기 때문입니다. 이럴 때는 사용자 정의 주입을 사용할 수 있습니다.

`config/dependence.php`(파일이 없으면 생성)에 다음 코드를 추가합니다:

```php
return [
    // ... 기타 설정 생략

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

의존성 주입에서 `app\service\Mailer` 인스턴스가 필요할 때 이 설정에서 만든 `app\service\Mailer` 인스턴스를 자동으로 사용합니다.

`config/dependence.php`에서 `new`로 `Mailer` 클래스를 인스턴스화합니다. 이 예시에서는 문제가 없지만, `Mailer`가 다른 클래스에 의존하거나 내부에서 속성 주입을 사용하는 경우 `new`로 초기화하면 의존성 자동 주입이 이루어지지 않습니다. 해결 방법은 사용자 정의 인터페이스 주입을 사용하고, `Container::get(클래스명)` 또는 `Container::make(클래스명, [생성자 인자])`로 클래스를 초기화하는 것입니다.

## 사용자 정의 인터페이스 주입

실제 프로젝트에서는 구체적인 클래스보다 인터페이스에 기반한 프로그래밍을 선호합니다. 예를 들어 `app\controller\UserController`는 `app\service\Mailer` 대신 `app\service\MailerInterface`를 사용해야 합니다.

`MailerInterface` 인터페이스를 정의합니다:

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

`MailerInterface`의 구현을 정의합니다:

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
        // 이메일 전송 코드 생략
    }
}
```

구체적인 구현 대신 `MailerInterface` 인터페이스를 사용합니다:

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

`config/dependence.php`에서 `MailerInterface` 구현을 다음과 같이 정의합니다:

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

비즈니스에서 `MailerInterface` 인터페이스가 필요할 때 자동으로 `Mailer` 구현이 사용됩니다.

> 인터페이스 지향 프로그래밍의 장점은 컴포넌트를 교체할 때 비즈니스 코드를 수정할 필요 없이 `config/dependence.php`의 구체적 구현만 변경하면 된다는 것입니다. 단위 테스트에도 매우 유용합니다.

## 기타 사용자 정의 주입

`config/dependence.php`는 클래스 의존성 외에 문자열, 숫자, 배열 등의 다른 값도 정의할 수 있습니다.

예를 들어 `config/dependence.php`를 다음과 같이 정의한 경우:

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

`#[Inject]`로 `smtp_host`와 `smtp_port`를 클래스 속성에 주입할 수 있습니다:

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
        // 이메일 전송 코드 생략
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // 192.168.1.11:25 출력
    }
}
```

# 지연 로딩(Lazy Loading)

> 지연 로딩은 객체의 생성 또는 초기화를 실제로 필요할 때까지 미루는 디자인 패턴입니다.

이 기능을 사용하려면 추가 의존성이 필요합니다. 다음 패키지는 `ocramius/proxy-manager`의 포크이며, 원본 저장소는 PHP 8을 지원하지 않습니다.

```
composer require friendsofphp/proxy-manager-lts
```

사용 방법:

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
        echo "MyClass 인스턴스화\n";
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
        echo "프록시 클래스명: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

출력:

```
프록시 클래스명: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass 인스턴스화
name: Lazy Loaded Object
```

위 예시는 `#[Injectable]` 속성이 선언된 클래스가 주입될 때 먼저 해당 클래스의 프록시 클래스가 생성되고, 임의의 메서드가 호출된 후에야 인스턴스화됨을 보여줍니다.

# 순환 의존성

순환 의존성은 여러 클래스가 서로 의존하여 폐쇄적인 의존 관계를 형성하는 것을 말합니다.

- 직접 순환 의존성
  - 모듈 A가 모듈 B에 의존하고, 모듈 B가 모듈 A에 의존
  - A → B → A 의존 루프 형성

- 간접 순환 의존성
  - 여러 모듈이 관련된 의존 루프
  - 예: A → B → C → A

속성 주입을 사용할 때 `php-di`는 순환 의존성을 자동 감지하고 예외를 던집니다. 필요 시 다음 코드처럼 사용하세요:

```php
class userController
{

    // 다음 코드 제거
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## 더 알아보기

[php-di 매뉴얼](https://php-di.org/doc/getting-started.html)을 참조하세요.
