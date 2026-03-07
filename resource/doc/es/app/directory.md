# Estructura de directorios

```
plugin/
└── foo
    ├── app
    │   ├── controller
    │   │   └── IndexController.php
    │   ├── exception
    │   │   └── Handler.php
    │   ├── functions.php
    │   ├── middleware
    │   ├── model
    │   └── view
    │       └── index
    │           └── index.html
    ├── config
    │   ├── app.php
    │   ├── autoload.php
    │   ├── container.php
    │   ├── database.php
    │   ├── exception.php
    │   ├── log.php
    │   ├── middleware.php
    │   ├── process.php
    │   ├── redis.php
    │   ├── route.php
    │   ├── static.php
    │   ├── thinkorm.php
    │   ├── translation.php
    │   └── view.php
    ├── public
    └── api
```

Un complemento de aplicación tiene la misma estructura de directorios y archivos de configuración que webman. En la práctica, la experiencia de desarrollo es prácticamente idéntica a la de una aplicación webman convencional.

Los directorios y la nomenclatura de los complementos siguen la especificación PSR-4. Al estar los complementos en el directorio `plugin`, todos los espacios de nombres comienzan por `plugin`, por ejemplo `plugin\foo\app\controller\UserController`.

## Sobre el directorio api

Cada complemento tiene un directorio `api`. Si tu aplicación ofrece interfaces internas para que otras aplicaciones las invoquen, colócalas en el directorio `api`.

Nota: aquí las interfaces son interfaces de llamada a funciones, no interfaces de red/HTTP.

Por ejemplo, el complemento de correo ofrece la interfaz `Email::send()` en `plugin/email/api/Email.php` para que otras aplicaciones envíen correos. Además, `plugin/email/api/Install.php` se genera automáticamente para que el mercado de complementos de webman-admin ejecute la instalación o desinstalación.
