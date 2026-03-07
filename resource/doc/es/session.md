# Gestión de sesiones webman

## Ejemplo
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

Obtenga una instancia de `Workerman\Protocols\Http\Session` mediante `$request->session();` y use sus métodos para añadir, modificar o eliminar datos de sesión.

> **Nota**
> Los datos de sesión se guardan automáticamente cuando se destruye el objeto de sesión.
> Si guarda el objeto de sesión en una variable global, impedirá su destrucción y no se guardará automáticamente. En ese caso debe llamar manualmente a `$session->save()` para guardar los datos.

## Obtener todos los datos de sesión
```php
$session = $request->session();
$all = $session->all();
```
Devuelve un array. Si no hay datos de sesión, devuelve un array vacío.


## Obtener un valor de la sesión
```php
$session = $request->session();
$name = $session->get('name');
```
Devuelve null si el dato no existe.

También puede pasar un valor predeterminado como segundo argumento al método `get`. Si no se encuentra el valor correspondiente en el array de sesión, se devuelve el valor predeterminado. Por ejemplo:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## Almacenar datos de sesión
Use el método `set` para almacenar un dato.
```php
$session = $request->session();
$session->set('name', 'tom');
```
El método `set` no devuelve nada. Los datos de sesión se guardan automáticamente cuando se destruye el objeto de sesión.

Use el método `put` para almacenar varios valores.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
De igual modo, el método `put` no devuelve nada.

## Eliminar datos de sesión
Use el método `forget` para eliminar uno o más datos de sesión.
```php
$session = $request->session();
// Eliminar uno
$session->forget('name');
// Eliminar varios
$session->forget(['name', 'age']);
```

El sistema también proporciona el método `delete`. A diferencia de `forget`, solo puede eliminar un dato.
```php
$session = $request->session();
// Equivalente a $session->forget('name');
$session->delete('name');
```


## Obtener y eliminar un valor de sesión
```php
$session = $request->session();
$name = $session->pull('name');
```
Tiene el mismo efecto que el siguiente código:
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
Devuelve null si el valor de sesión correspondiente no existe.


## Eliminar todos los datos de sesión
```php
$request->session()->flush();
```
No devuelve nada. Los datos de sesión se eliminan automáticamente del almacenamiento cuando se destruye el objeto de sesión.


## Comprobar si existe un valor de sesión
```php
$session = $request->session();
$has = $session->has('name');
```
Devuelve false si el valor de sesión no existe o es null; en caso contrario, devuelve true.

```php
$session = $request->session();
$has = $session->exists('name');
```
El código anterior también comprueba si existe un valor de sesión. La diferencia es que `exists` devuelve true incluso cuando el valor es null.

## Función auxiliar session()

webman proporciona la función auxiliar `session()` para realizar las mismas operaciones.
```php
// Obtener la instancia de sesión
$session = session();
// Equivalente a
$session = $request->session();

// Obtener un valor
$value = session('key', 'default');
// Equivalente a
$value = session()->get('key', 'default');
// Equivalente a
$value = $request->session()->get('key', 'default');

// Asignar valores a la sesión
session(['key1'=>'value1', 'key2' => 'value2']);
// Equivalente a
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// Equivalente a
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## Archivo de configuración
El archivo de configuración de sesión está en `config/session.php`. Su contenido es similar a:
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class o RedisSessionHandler::class o RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // Si handler es FileSessionHandler::class, el valor es 'file',
    // si handler es RedisSessionHandler::class, el valor es 'redis'
    // si handler es RedisClusterSessionHandler::class, el valor es 'redis_cluster' (clúster Redis)
    'type'    => 'file',

    // Cada handler utiliza una configuración distinta
    'config' => [
        // Configuración cuando type es 'file'
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // Configuración cuando type es 'redis'
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // Nombre de la cookie para almacenar el session_id
    'auto_update_timestamp' => false,  // Actualizar automáticamente la sesión, por defecto: desactivado
    'lifetime' => 7*24*60*60,          // Tiempo de expiración de la sesión
    'cookie_lifetime' => 365*24*60*60, // Tiempo de expiración de la cookie que almacena el session_id
    'cookie_path' => '/',              // Ruta de la cookie que almacena el session_id
    'domain' => '',                    // Dominio de la cookie que almacena el session_id
    'http_only' => true,               // Habilitar httpOnly, por defecto: activado
    'secure' => false,                 // Habilitar sesión solo en HTTPS, por defecto: desactivado
    'same_site' => '',                 // Prevención de ataques CSRF y seguimiento de usuarios, opciones: strict/lax/none
    'gc_probability' => [1, 1000],     // Probabilidad de recolección de sesión
];
```

## Seguridad
No se recomienda almacenar directamente instancias de clases en la sesión, especialmente de clases de fuentes no confiables. La deserialización puede suponer riesgos de seguridad.

