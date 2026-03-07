# Manejo de eventos
`webman/event` proporciona un mecanismo de eventos elegante que permite ejecutar lógica de negocio sin modificar el código, logrando el desacoplamiento entre módulos. Un escenario típico: cuando un nuevo usuario se registra correctamente, basta con publicar un evento personalizado como `user.register`, y cada módulo podrá recibir ese evento y ejecutar la lógica correspondiente.

## Instalación
`composer require webman/event`

## Suscribirse a eventos
La suscripción a eventos se configura de forma unificada mediante el archivo `config/event.php`.

```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...otras funciones de manejo de eventos...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...otras funciones de manejo de eventos...
    ]
];
```

**Nota:**
- `user.register`, `user.logout`, etc., son nombres de eventos (tipo cadena). Se recomienda usar palabras en minúsculas separadas por punto (`.`).
- Un evento puede tener varias funciones de manejo; se invocan en el orden configurado.

## Funciones de manejo de eventos
Las funciones de manejo pueden ser métodos de clase, funciones o closures.

Por ejemplo, cree la clase de manejo `app/event/User.php` (cree el directorio si no existe):

```php
<?php
namespace app\event;
class User
{
    function register($user)
    {
        var_export($user);
    }
 
    function logout($user)
    {
        var_export($user);
    }
}
```

## Publicar eventos
Use `Event::dispatch($event_name, $data);` o `Event::emit($event_name, $data);` para publicar un evento. Por ejemplo:

```php
<?php
namespace app\controller;
use support\Request;
use Webman\Event\Event;
class User
{
    public function register(Request $request)
    {
        $user = [
            'name' => 'webman',
            'age' => 2
        ];
        Event::dispatch('user.register', $user);
    }
}
```

Hay dos funciones para publicar: `Event::dispatch($event_name, $data);` y `Event::emit($event_name, $data);`, ambas con los mismos parámetros. La diferencia: `emit` captura excepciones internamente; si una función lanza una excepción, las demás siguen ejecutándose. `dispatch` no captura excepciones; si alguna función lanza una excepción, se detiene la ejecución y la excepción se propaga hacia arriba.

> **Sugerencia**
> El parámetro `$data` puede ser cualquier tipo de datos (array, instancia de clase, cadena, etc.).

## Escucha de eventos con comodines
La escucha con comodines permite manejar varios eventos con el mismo listener. Por ejemplo, en `config/event.php`:

```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```

Puede obtener el nombre concreto del evento mediante el segundo parámetro `$event_data` de la función de manejo:

```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // nombre concreto del evento, ej. user.register, user.logout, etc.
        var_export($user);
    }
}
```

## Detener la difusión del evento
Si una función de manejo devuelve `false`, se detiene la difusión de ese evento.

## Manejo de eventos con closures
La función de manejo puede ser un método de clase o una closure. Por ejemplo:

```php
<?php
return [
    'user.login' => [
        function($user){
            var_dump($user);
        }
    ]
];
```

## Ver eventos y listeners
Use el comando `php webman event:list` para ver todos los eventos y listeners configurados en el proyecto.

## Ámbito de soporte
Además del proyecto principal, los [plugins base](../plugin/base.md) y los [plugins de aplicación](../app/app.md) también admiten la configuración en `event.php`.
**Configuración de plugin base:** `config/plugin/vendor/nombre-plugin/event.php`
**Configuración de plugin de aplicación:** `plugin/nombre-plugin/config/event.php`

## Consideraciones
El manejo de eventos no es asíncrono. No es adecuado para lógica de negocio lenta; esta debería gestionarse mediante colas de mensajes, como [webman/redis-queue](https://www.workerman.net/plugin/12).
