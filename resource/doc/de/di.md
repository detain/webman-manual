# Abhängigkeits-Automatik-Injektion

In webman ist die automatische Abhängigkeitsinjektion eine optionale Funktion und standardmäßig deaktiviert. Wenn Sie die automatische Abhängigkeitsinjektion benötigen, empfehlen wir die Verwendung von [php-di](https://php-di.org/doc/getting-started.html). Im Folgenden wird die Verwendung von `php-di` in Verbindung mit webman beschrieben.

## Installation

```
composer require php-di/php-di:^7.0
```

Ändern Sie die Konfiguration `config/container.php`. Der finale Inhalt sollte wie folgt aussehen:

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> Die Datei `config/container.php` muss letztlich eine Container-Instanz zurückgeben, die der PSR-11-Spezifikation entspricht. Wenn Sie `php-di` nicht verwenden möchten, können Sie hier eine andere PSR-11-konforme Container-Instanz erstellen und zurückgeben. Die Standardkonfiguration bietet nur die grundlegenden Webman-Container-Funktionen.

## Konstruktorinjektion

Erstellen Sie die neue Datei `app/service/Mailer.php` (erstellen Sie das Verzeichnis bei Bedarf) mit folgendem Inhalt:

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // E-Mail-Versandcode ausgelassen
    }
}
```

Der Inhalt von `app/controller/UserController.php` lautet wie folgt:

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
        $this->mailer->mail('hello@webman.com', 'Hallo und willkommen!');
        return response('ok');
    }
}
```

Normalerweise wäre folgender Code erforderlich, um `app\controller\UserController` zu instanziieren:

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

Bei Verwendung von `php-di` müssen Entwickler das `Mailer` im Controller nicht mehr manuell instanziieren – webman erledigt dies automatisch. Wenn bei der Instanziierung von `Mailer` weitere Klassenabhängigkeiten bestehen, werden diese ebenfalls automatisch instanziiert und injiziert. Der Entwickler benötigt keine Initialisierungsarbeiten.

> **Hinweis**
> Nur von Framework oder `php-di` erstellte Instanzen unterstützen die automatische Abhängigkeitsinjektion. Mit `new` manuell erstellte Instanzen können keine automatische Abhängigkeitsinjektion nutzen. Wenn Injektion erforderlich ist, verwenden Sie die `support\Container`-Schnittstelle anstelle von `new`, z. B.:

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// Mit new erstellte Instanzen können keine Abhängigkeitsinjektion nutzen
$user_service = new UserService;
// Mit new erstellte Instanzen können keine Abhängigkeitsinjektion nutzen
$log_service = new LogService($path, $name);

// Mit Container erstellte Instanzen können Abhängigkeitsinjektion nutzen
$user_service = Container::get(UserService::class);
// Mit Container erstellte Instanzen können Abhängigkeitsinjektion nutzen
$log_service = Container::make(LogService::class, [$path, $name]);
```

## Attribut-Injektion

Neben der Konstruktor-Abhängigkeitsinjektion können Sie auch Attribut-Injektion verwenden. Fortsetzung des vorherigen Beispiels – ändern Sie `app\controller\UserController` wie folgt:

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
        $this->mailer->mail('hello@webman.com', 'Hallo und willkommen!');
        return response('ok');
    }
}
```

Dieses Beispiel verwendet das `#[Inject]`-Attribut zur Injektion und injiziert die Instanz automatisch in die Member-Variable basierend auf dem Objekttyp. Der Effekt entspricht der Konstruktorinjektion, der Code ist jedoch prägnanter.

> **Hinweis**
> Webman unterstützt Controller-Parameterinjektion nicht vor Version 1.4.6. Beispielsweise wird der folgende Code bei webman<=1.4.6 nicht unterstützt:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // Controller-Parameterinjektion wird vor Version 1.4.6 nicht unterstützt
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'Hallo und willkommen!');
        return response('ok');
    }
}
```

## Benutzerdefinierte Konstruktorinjektion

Manchmal sind die an den Konstruktor übergebenen Parameter keine Klasseninstanzen, sondern Zeichenfolgen, Zahlen, Arrays oder andere Nicht-Objekt-Daten. Beispielsweise benötigt der Mailer-Konstruktor die SMTP-Server-IP und den Port:

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
        // E-Mail-Versandcode ausgelassen
    }
}
```

In diesem Fall kann die Konstruktor-Auto-Injektion nicht direkt verwendet werden, da `php-di` die Werte von `$smtp_host` und `$smtp_port` nicht ermitteln kann. In solchen Fällen kann eine benutzerdefinierte Injektion verwendet werden.

Fügen Sie folgenden Code in `config/dependence.php` ein (die Datei erstellen, falls sie nicht existiert):

```php
return [
    // ... Weitere Konfigurationen ausgelassen

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

Wenn die Abhängigkeitsinjektion eine Instanz von `app\service\Mailer` benötigt, wird automatisch die in dieser Konfiguration erstellte Instanz verwendet.

Beachten Sie, dass `config/dependence.php` `new` zur Instanziierung der `Mailer`-Klasse verwendet. Dies ist in diesem Beispiel unproblematisch. Wenn die `Mailer`-Klasse jedoch von anderen Klassen abhängt oder intern Attribut-Injektion nutzt, führt die Initialisierung mit `new` zu keiner automatischen Abhängigkeitsinjektion. Die Lösung ist die benutzerdefinierte Schnittstelleninjektion mit `Container::get(Klassenname)` oder `Container::make(Klassenname, [Konstruktorparameter])`.

## Benutzerdefinierte Schnittstelleninjektion

In realen Projekten ist die Programmierung gegen Schnittstellen statt konkreter Klassen sinnvoller. Beispielsweise sollte `app\controller\UserController` `app\service\MailerInterface` statt `app\service\Mailer` verwenden.

Definieren Sie die Schnittstelle `MailerInterface`:

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

Definieren Sie die Implementierung von `MailerInterface`:

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
        // E-Mail-Versandcode ausgelassen
    }
}
```

Verwenden Sie `MailerInterface` statt der konkreten Implementierung:

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
        $this->mailer->mail('hello@webman.com', 'Hallo und willkommen!');
        return response('ok');
    }
}
```

Definieren Sie die Implementierung von `MailerInterface` in `config/dependence.php`:

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

Wenn die Anwendung die Schnittstelle `MailerInterface` benötigt, wird automatisch die `Mailer`-Implementierung verwendet.

> Der Vorteil der Programmierung gegen Schnittstellen: Beim Austausch einer Komponente muss der Geschäftslogik-Code nicht geändert werden – nur die konkrete Implementierung in `config/dependence.php`. Dies ist auch für Unit-Tests sehr nützlich.

## Weitere benutzerdefinierte Injektionen

Zusätzlich zur Definition von Klassenabhängigkeiten kann `config/dependence.php` auch andere Werte wie Zeichenfolgen, Zahlen und Arrays definieren.

Beispielsweise, wenn `config/dependence.php` wie folgt definiert ist:

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

 können Sie `smtp_host` und `smtp_port` mit `#[Inject]` in Klassen-Eigenschaften injizieren:

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
        // E-Mail-Versandcode ausgelassen
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // Gibt 192.168.1.11:25 aus
    }
}
```

# Lazy Loading

> Lazy Loading ist ein Entwurfsmuster, das die Erstellung oder Initialisierung von Objekten verzögert, bis sie tatsächlich benötigt werden.

Diese Funktion erfordert eine zusätzliche Abhängigkeit. Das folgende Paket ist ein Fork von `ocramius/proxy-manager`; das ursprüngliche Repository unterstützt PHP 8 nicht.

```
composer require friendsofphp/proxy-manager-lts
```

Verwendung:

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
        echo "MyClass instanziiert\n";
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
        echo "Proxy-Klassenname: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

Ausgabe:

```
Proxy-Klassenname: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass instanziiert
name: Lazy Loaded Object
```

Dieses Beispiel zeigt: Bei Klassen mit dem `#[Injectable]`-Attribut wird beim Injizieren zuerst eine Proxy-Klasse erstellt. Die eigentliche Klasse wird erst instanziiert, wenn eine ihrer Methoden aufgerufen wird.

# Zirkuläre Abhängigkeiten

Zirkuläre Abhängigkeiten liegen vor, wenn mehrere Klassen voneinander abhängen und eine geschlossene Abhängigkeitsschleife bilden.

- Direkte zirkuläre Abhängigkeit
  - Modul A hängt von Modul B ab, Modul B hängt von Modul A ab
  - Bildet den Zyklus: A → B → A

- Indirekte zirkuläre Abhängigkeit
  - Mehrere Module sind in einem Abhängigkeitszyklus involviert
  - Z. B.: A → B → C → A

Bei Verwendung der Attribut-Injektion erkennt `php-di` zirkuläre Abhängigkeiten automatisch und wirft eine Ausnahme. Bei Bedarf können Sie folgendermaßen vorgehen:

```php
class userController
{

    // Folgenden Code entfernen
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## Weitere Informationen

Bitte beachten Sie die [php-di-Dokumentation](https://php-di.org/doc/getting-started.html).
