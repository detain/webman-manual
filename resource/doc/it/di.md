# Iniezione automatica delle dipendenze

In webman l'iniezione automatica delle dipendenze è una funzionalità opzionale e disabilitata di default. Se ti serve l'iniezione automatica delle dipendenze, si consiglia di usare [php-di](https://php-di.org/doc/getting-started.html). Di seguito viene descritto l'uso di `php-di` con webman.

## Installazione

```
composer require php-di/php-di:^7.0
```

Modifica la configurazione `config/container.php`. Il contenuto finale deve essere il seguente:

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> Il file `config/container.php` deve restituire un'istanza di contenitore conforme alla specifica PSR-11. Se non vuoi usare `php-di`, puoi creare e restituire qui un'altra istanza di contenitore conforme a PSR-11. La configurazione predefinita fornisce solo le funzionalità base del contenitore webman.

## Iniezione per costruttore

Crea il file `app/service/Mailer.php` (crea la directory se non esiste) con il seguente contenuto:

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // Codice invio email omesso
    }
}
```

Il contenuto di `app/controller/UserController.php` è il seguente:

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
        $this->mailer->mail('hello@webman.com', 'Ciao e benvenuto!');
        return response('ok');
    }
}
```

Normalmente, per istanziare `app\controller\UserController` sarebbe necessario il seguente codice:

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

Usando `php-di`, gli sviluppatori non devono istanziare manualmente `Mailer` nel controller — webman lo fa automaticamente. Se durante l'istanziazione di `Mailer` ci sono altre dipendenze di classi, webman le istanzierà e le inietterà automaticamente. Non è richiesto alcun lavoro di inizializzazione allo sviluppatore.

> **Nota**
> Solo le istanze create dal framework o da `php-di` supportano l'iniezione automatica delle dipendenze. Le istanze create manualmente con `new` non possono usarla. Per iniettare, usa l'interfaccia `support\Container` al posto di `new`, ad esempio:

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// Le istanze create con new non possono usare l'iniezione delle dipendenze
$user_service = new UserService;
// Le istanze create con new non possono usare l'iniezione delle dipendenze
$log_service = new LogService($path, $name);

// Le istanze create con Container possono usare l'iniezione delle dipendenze
$user_service = Container::get(UserService::class);
// Le istanze create con Container possono usare l'iniezione delle dipendenze
$log_service = Container::make(LogService::class, [$path, $name]);
```

## Iniezione per attributi

Oltre all'iniezione per costruttore, puoi usare l'iniezione per attributi. Continuando l'esempio precedente, modifica `app\controller\UserController` così:

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
        $this->mailer->mail('hello@webman.com', 'Ciao e benvenuto!');
        return response('ok');
    }
}
```

Questo esempio usa l'attributo `#[Inject]` per l'iniezione e inietta automaticamente l'istanza nella variabile membro in base al tipo di oggetto. L'effetto è lo stesso dell'iniezione per costruttore, ma il codice è più conciso.

> **Nota**
> webman non supporta l'iniezione dei parametri del controller prima della versione 1.4.6. Ad esempio, il seguente codice non è supportato quando webman<=1.4.6:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // L'iniezione dei parametri del controller non è supportata prima della versione 1.4.6
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'Ciao e benvenuto!');
        return response('ok');
    }
}
```

## Iniezione personalizzata per costruttore

A volte i parametri passati al costruttore non sono istanze di classi ma stringhe, numeri, array o altri dati non oggetto. Ad esempio, il costruttore di Mailer potrebbe aver bisogno dell'IP e della porta del server SMTP:

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
        // Codice invio email omesso
    }
}
```

In questo caso non si può usare direttamente l'iniezione automatica per costruttore, perché `php-di` non può determinare i valori di `$smtp_host` e `$smtp_port`. In tal caso si può provare l'iniezione personalizzata.

Aggiungi il seguente codice in `config/dependence.php` (crea il file se non esiste):

```php
return [
    // ... Altre configurazioni omesse

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

Quando l'iniezione delle dipendenze deve ottenere un'istanza di `app\service\Mailer`, userà automaticamente l'istanza creata in questa configurazione.

Nota che `config/dependence.php` usa `new` per istanziare la classe `Mailer`. In questo esempio non è un problema, ma se la classe `Mailer` dipende da altre classi o usa l'iniezione per attributi internamente, l'inizializzazione con `new` non eseguirà l'iniezione automatica delle dipendenze. La soluzione è usare l'iniezione personalizzata per interfaccia e inizializzare le classi tramite `Container::get(nome classe)` o `Container::make(nome classe, [parametri costruttore])`.

## Iniezione personalizzata per interfaccia

Nei progetti reali è preferibile programmare contro le interfacce piuttosto che contro classi concrete. Ad esempio, `app\controller\UserController` dovrebbe dipendere da `app\service\MailerInterface` invece che da `app\service\Mailer`.

Definisci l'interfaccia `MailerInterface`:

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

Definisci l'implementazione di `MailerInterface`:

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
        // Codice invio email omesso
    }
}
```

Usa l'interfaccia `MailerInterface` invece dell'implementazione concreta:

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
        $this->mailer->mail('hello@webman.com', 'Ciao e benvenuto!');
        return response('ok');
    }
}
```

Definisci l'implementazione di `MailerInterface` in `config/dependence.php`:

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

Quando l'applicazione deve usare l'interfaccia `MailerInterface`, verrà usata automaticamente l'implementazione `Mailer`.

> Il vantaggio di programmare contro le interfacce è che quando serve sostituire un componente non è necessario modificare il codice di business, solo l'implementazione concreta in `config/dependence.php`. È molto utile anche per i test unitari.

## Altre iniezioni personalizzate

Oltre alle dipendenze di classi, `config/dependence.php` può definire anche altri valori come stringhe, numeri, array, ecc.

Ad esempio, se `config/dependence.php` è definito così:

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

Puoi iniettare `smtp_host` e `smtp_port` nelle proprietà della classe con `#[Inject]`:

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
        // Codice invio email omesso
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // Mostrerà 192.168.1.11:25
    }
}
```

# Caricamento differito (Lazy Loading)

> Il caricamento differito è un pattern di progettazione che ritarda la creazione o l'inizializzazione degli oggetti finché non sono effettivamente necessari.

Questa funzionalità richiede una dipendenza aggiuntiva. Il seguente pacchetto è un fork di `ocramius/proxy-manager`; il repository originale non supporta PHP 8.

```
composer require friendsofphp/proxy-manager-lts
```

Utilizzo:

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
        echo "MyClass istanziato\n";
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
        echo "Nome classe proxy: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

Output:

```
Nome classe proxy: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass istanziato
name: Lazy Loaded Object
```

Questo esempio mostra che per una classe dichiarata con l'attributo `#[Injectable]`, quando viene iniettata viene prima creata la classe proxy. La classe reale viene istanziata solo quando viene chiamato uno dei suoi metodi.

# Dipendenze circolari

Le dipendenze circolari si verificano quando più classi dipendono l'una dall'altra, formando un ciclo chiuso di dipendenze.

- Dipendenza circolare diretta
  - Il modulo A dipende dal modulo B e il modulo B dipende dal modulo A
  - Forma il ciclo: A → B → A

- Dipendenza circolare indiretta
  - Coinvolge più moduli in un ciclo di dipendenze
  - Ad esempio: A → B → C → A

Quando si usa l'iniezione per attributi, `php-di` rileva automaticamente le dipendenze circolari e solleva un'eccezione. Se necessario, usa il seguente approccio:

```php
class userController
{

    // Rimuovi questo codice
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## Ulteriori informazioni

Consulta la [documentazione php-di](https://php-di.org/doc/getting-started.html).
