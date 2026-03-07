# Автоматическое внедрение зависимостей

В webman автоматическое внедрение зависимостей — опциональная функция, по умолчанию отключена. Если оно вам нужно, рекомендуется использовать [php-di](https://php-di.org/doc/getting-started.html). Ниже описан способ использования `php-di` с webman.

## Установка

```
composer require php-di/php-di:^7.0
```

Измените конфигурацию `config/container.php`. Итоговое содержимое должно быть таким:

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> Файл `config/container.php` должен возвращать экземпляр контейнера, соответствующий спецификации PSR-11. Если вы не хотите использовать `php-di`, можете создать и вернуть здесь другой экземпляр контейнера, совместимый с PSR-11. Конфигурация по умолчанию предоставляет только базовую функциональность контейнера webman.

## Внедрение через конструктор

Создайте файл `app/service/Mailer.php` (создайте каталог при необходимости) со следующим содержимым:

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // Код отправки письма опущен
    }
}
```

Содержимое `app/controller/UserController.php`:

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
        $this->mailer->mail('hello@webman.com', 'Привет и добро пожаловать!');
        return response('ok');
    }
}
```

Обычно для инстанцирования `app\controller\UserController` потребовался бы следующий код:

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

При использовании `php-di` разработчикам не нужно вручную инстанцировать `Mailer` в контроллере — webman сделает это автоматически. Если при инстанцировании `Mailer` есть другие зависимости, webman также инстанцирует и внедрит их. От разработчика не требуется никакой инициализации.

> **Примечание**
> Автоматическое внедрение зависимостей поддерживают только экземпляры, созданные фреймворком или `php-di`. Экземпляры, созданные вручную через `new`, не поддерживают его. Для внедрения используйте интерфейс `support\Container` вместо `new`, например:

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// Экземпляры, созданные через new, не поддерживают внедрение зависимостей
$user_service = new UserService;
// Экземпляры, созданные через new, не поддерживают внедрение зависимостей
$log_service = new LogService($path, $name);

// Экземпляры, созданные через Container, поддерживают внедрение зависимостей
$user_service = Container::get(UserService::class);
// Экземпляры, созданные через Container, поддерживают внедрение зависимостей
$log_service = Container::make(LogService::class, [$path, $name]);
```

## Внедрение через атрибуты

Помимо внедрения через конструктор, можно использовать внедрение через атрибуты. Продолжая предыдущий пример, измените `app\controller\UserController` так:

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
        $this->mailer->mail('hello@webman.com', 'Привет и добро пожаловать!');
        return response('ok');
    }
}
```

В этом примере для внедрения используется атрибут `#[Inject]`, а экземпляр автоматически внедряется в член класса по типу объекта. Эффект такой же, как при внедрении через конструктор, но код компактнее.

> **Примечание**
> webman не поддерживает внедрение параметров контроллера до версии 1.4.6. Например, следующий код не поддерживается при webman<=1.4.6:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // Внедрение параметров контроллера не поддерживается до версии 1.4.6
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'Привет и добро пожаловать!');
        return response('ok');
    }
}
```

## Пользовательское внедрение через конструктор

Иногда параметры конструктора могут быть не экземплярами классов, а строками, числами, массивами и другими необъектными данными. Например, конструктор Mailer может требовать IP и порт SMTP-сервера:

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
        // Код отправки письма опущен
    }
}
```

В этом случае прямое внедрение через конструктор недоступно, поскольку `php-di` не может определить значения `$smtp_host` и `$smtp_port`. В таком случае можно использовать пользовательское внедрение.

Добавьте следующий код в `config/dependence.php` (создайте файл при отсутствии):

```php
return [
    // ... Остальная конфигурация опущена

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

Когда для внедрения зависимостей потребуется экземпляр `app\service\Mailer`, будет автоматически использован экземпляр, созданный в этой конфигурации.

Обратите внимание, что `config/dependence.php` использует `new` для инстанцирования класса `Mailer`. В этом примере это допустимо, но если класс `Mailer` зависит от других классов или использует внутри внедрение через атрибуты, инициализация через `new` не выполнит автоматическое внедрение зависимостей. Решение — пользовательское внедрение через интерфейс и инициализация классов через `Container::get(имя класса)` или `Container::make(имя класса, [параметры конструктора])`.

## Пользовательское внедрение через интерфейс

В реальных проектах предпочтительнее программировать против интерфейсов, а не конкретных классов. Например, `app\controller\UserController` должен зависеть от `app\service\MailerInterface`, а не от `app\service\Mailer`.

Определите интерфейс `MailerInterface`:

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

Определите реализацию `MailerInterface`:

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
        // Код отправки письма опущен
    }
}
```

Используйте интерфейс `MailerInterface` вместо конкретной реализации:

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
        $this->mailer->mail('hello@webman.com', 'Привет и добро пожаловать!');
        return response('ok');
    }
}
```

Определите реализацию `MailerInterface` в `config/dependence.php`:

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

Когда приложению понадобится интерфейс `MailerInterface`, будет автоматически использована реализация `Mailer`.

> Преимущество программирования против интерфейсов в том, что при замене компонента не нужно менять бизнес-код — только конкретную реализацию в `config/dependence.php`. Это также полезно для модульного тестирования.

## Другие пользовательские внедрения

Помимо зависимостей классов, `config/dependence.php` может определять и другие значения: строки, числа, массивы и т.д.

Например, если `config/dependence.php` определён так:

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

можно внедрить `smtp_host` и `smtp_port` в свойства класса через `#[Inject]`:

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
        // Код отправки письма опущен
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // Выведет 192.168.1.11:25
    }
}
```

# Отложенная загрузка (Lazy Loading)

> Отложенная загрузка — шаблон проектирования, при котором создание или инициализация объектов откладывается до момента фактического использования.

Для этой функции требуется дополнительная зависимость. Следующий пакет — форк `ocramius/proxy-manager`; оригинальный репозиторий не поддерживает PHP 8.

```
composer require friendsofphp/proxy-manager-lts
```

Использование:

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
        echo "MyClass инстанцирован\n";
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
        echo "Имя класса-прокси: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

Вывод:

```
Имя класса-прокси: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass инстанцирован
name: Lazy Loaded Object
```

В этом примере видно, что для класса с атрибутом `#[Injectable]` при внедрении сначала создаётся класс-прокси. Реальный класс инстанцируется только при вызове любого из его методов.

# Циклические зависимости

Циклические зависимости возникают, когда несколько классов зависят друг от друга, образуя замкнутый цикл зависимостей.

- Прямая циклическая зависимость
  - Модуль A зависит от модуля B, модуль B зависит от модуля A
  - Образует цикл: A → B → A

- Косвенная циклическая зависимость
  - В цикл зависимостей входит несколько модулей
  - Например: A → B → C → A

При использовании внедрения через атрибуты `php-di` автоматически обнаруживает циклические зависимости и выбрасывает исключение. При необходимости используйте следующий подход:

```php
class userController
{

    // Удалите этот код
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## Дополнительная информация

Обратитесь к [документации php-di](https://php-di.org/doc/getting-started.html).
