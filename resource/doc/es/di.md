# Inyección automática de dependencias

En webman, la inyección automática de dependencias es una funcionalidad opcional que está desactivada por defecto. Si necesitas inyección automática de dependencias, se recomienda usar [php-di](https://php-di.org/doc/getting-started.html). A continuación se describe el uso de `php-di` con webman.

## Instalación

```
composer require php-di/php-di:^7.0
```

Modifica la configuración `config/container.php`. El contenido final debe ser el siguiente:

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> El archivo `config/container.php` debe devolver finalmente una instancia de contenedor conforme a la especificación PSR-11. Si no deseas usar `php-di`, puedes crear y devolver aquí otra instancia de contenedor conforme a PSR-11. La configuración por defecto solo proporciona la funcionalidad básica del contenedor de webman.

## Inyección por constructor

Crea el archivo `app/service/Mailer.php` (crea el directorio si no existe) con el siguiente contenido:

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // Código de envío de correo omitido
    }
}
```

El contenido de `app/controller/UserController.php` es el siguiente:

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
        $this->mailer->mail('hello@webman.com', '¡Hola y bienvenido!');
        return response('ok');
    }
}
```

Normalmente, el siguiente código sería necesario para instanciar `app\controller\UserController`:

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

Al usar `php-di`, los desarrolladores no necesitan instanciar manualmente `Mailer` en el controlador; webman lo hace automáticamente. Si durante la instanciación de `Mailer` hay otras dependencias de clases, webman también las instanciará e inyectará automáticamente. El desarrollador no requiere ningún trabajo de inicialización.

> **Nota**
> Solo las instancias creadas por el framework o `php-di` permiten la inyección automática de dependencias. Las instancias creadas manualmente con `new` no pueden utilizarla. Para inyectar, usa la interfaz `support\Container` en lugar de `new`, por ejemplo:

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// Las instancias creadas con new no pueden usar inyección de dependencias
$user_service = new UserService;
// Las instancias creadas con new no pueden usar inyección de dependencias
$log_service = new LogService($path, $name);

// Las instancias creadas con Container pueden usar inyección de dependencias
$user_service = Container::get(UserService::class);
// Las instancias creadas con Container pueden usar inyección de dependencias
$log_service = Container::make(LogService::class, [$path, $name]);
```

## Inyección por atributos

Además de la inyección por constructor, puedes usar inyección por atributos. Continuando el ejemplo anterior, modifica `app\controller\UserController` así:

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
        $this->mailer->mail('hello@webman.com', '¡Hola y bienvenido!');
        return response('ok');
    }
}
```

Este ejemplo usa el atributo `#[Inject]` para inyectar y coloca automáticamente la instancia en la variable miembro según el tipo de objeto. El efecto es el mismo que la inyección por constructor, pero el código es más conciso.

> **Nota**
> webman no admite la inyección de parámetros de controlador antes de la versión 1.4.6. Por ejemplo, el siguiente código no está soportado cuando webman<=1.4.6:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // La inyección de parámetros de controlador no está soportada antes de la versión 1.4.6
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', '¡Hola y bienvenido!');
        return response('ok');
    }
}
```

## Inyección personalizada por constructor

A veces los parámetros del constructor no son instancias de clase sino cadenas, números, arrays u otros datos no objeto. Por ejemplo, el constructor de Mailer puede necesitar la IP y el puerto del servidor SMTP:

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
        // Código de envío de correo omitido
    }
}
```

En este caso no se puede usar directamente la inyección automática por constructor, porque `php-di` no puede determinar los valores de `$smtp_host` y `$smtp_port`. En tal caso, se puede usar inyección personalizada.

Añade el siguiente código en `config/dependence.php` (crea el archivo si no existe):

```php
return [
    // ... Otras configuraciones omitidas

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

Cuando la inyección de dependencias necesite obtener una instancia de `app\service\Mailer`, usará automáticamente la instancia definida en esta configuración.

Observa que `config/dependence.php` usa `new` para instanciar la clase `Mailer`. En este ejemplo no hay problema, pero si la clase `Mailer` depende de otras clases o usa inyección por atributos internamente, la inicialización con `new` no realizará la inyección automática de dependencias. La solución es usar inyección personalizada por interfaz e inicializar las clases mediante `Container::get(nombre de clase)` o `Container::make(nombre de clase, [parámetros del constructor])`.

## Inyección personalizada por interfaz

En proyectos reales, es preferible programar contra interfaces en lugar de clases concretas. Por ejemplo, `app\controller\UserController` debería depender de `app\service\MailerInterface` en lugar de `app\service\Mailer`.

Define la interfaz `MailerInterface`:

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

Define la implementación de `MailerInterface`:

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
        // Código de envío de correo omitido
    }
}
```

Usa la interfaz `MailerInterface` en lugar de la implementación concreta:

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
        $this->mailer->mail('hello@webman.com', '¡Hola y bienvenido!');
        return response('ok');
    }
}
```

Define la implementación de `MailerInterface` en `config/dependence.php`:

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

Cuando la aplicación necesite usar la interfaz `MailerInterface`, se usará automáticamente la implementación `Mailer`.

> La ventaja de programar contra interfaces es que al reemplazar un componente no hay que cambiar el código de negocio, solo la implementación concreta en `config/dependence.php`. Esto también es muy útil para pruebas unitarias.

## Otras inyecciones personalizadas

Además de dependencias de clases, `config/dependence.php` puede definir otros valores como cadenas, números, arrays, etc.

Por ejemplo, si `config/dependence.php` se define así:

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

Puedes inyectar `smtp_host` y `smtp_port` en las propiedades de la clase con `#[Inject]`:

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
        // Código de envío de correo omitido
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // Mostrará 192.168.1.11:25
    }
}
```

# Carga diferida (Lazy Loading)

> La carga diferida es un patrón de diseño que retrasa la creación o inicialización de objetos hasta que sean realmente necesarios.

Esta funcionalidad requiere una dependencia adicional. El siguiente paquete es un fork de `ocramius/proxy-manager`; el repositorio original no soporta PHP 8.

```
composer require friendsofphp/proxy-manager-lts
```

Uso:

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
        echo "MyClass instanciado\n";
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
        echo "Nombre de clase proxy: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

Salida:

```
Nombre de clase proxy: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass instanciado
name: Lazy Loaded Object
```

Este ejemplo muestra que para una clase declarada con el atributo `#[Injectable]`, cuando se inyecta primero se crea la clase proxy. La clase real solo se instancia cuando se invoca cualquiera de sus métodos.

# Dependencias circulares

Las dependencias circulares ocurren cuando varias clases dependen unas de otras, formando un bucle cerrado de dependencias.

- Dependencia circular directa
  - El módulo A depende del módulo B y el módulo B depende del módulo A
  - Forma el ciclo: A → B → A

- Dependencia circular indirecta
  - Implica varios módulos en un ciclo de dependencias
  - Por ejemplo: A → B → C → A

Cuando se usa inyección por atributos, `php-di` detecta automáticamente dependencias circulares y lanza una excepción. Si es necesario, usa el siguiente enfoque:

```php
class userController
{

    // Eliminar este código
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## Más información

Consulta la [documentación de php-di](https://php-di.org/doc/getting-started.html).
