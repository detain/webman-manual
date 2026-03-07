# think-cache

think-cache es un componente extraído del framework thinkphp, con soporte de agrupación de conexiones añadido. Soporta automáticamente entornos de corrutinas y no corrutinas.

## Instalación
`composer require -W webman/think-cache`

Se requiere reiniciar (restart) después de la instalación (reload no tiene efecto)

### Archivo de configuración

El archivo de configuración es `config/think-cache.php`

## Uso

  ```php
  <?php
  namespace app\controller;
    
  use support\Request;
  use support\think\Cache;
  
  class UserController
  {
      public function db(Request $request)
      {
          $key = 'test_key';
          Cache::set($key, rand());
          return response(Cache::get($key));
      }
  }
  ```
## API proporcionada
```php
// Establecer caché
Cache::set('val','value',600);
// Comprobar si existe el caché
Cache::has('val');
// Obtener caché
Cache::get('val');
// Eliminar caché
Cache::delete('val');
// Limpiar caché
Cache::clear();
// Leer y eliminar caché
Cache::pull('val');
// Escribir si no existe
Cache::remember('val',10);

// Para datos de caché numéricos
// Incrementar caché en 1
Cache::inc('val');
// Incrementar caché en 5
Cache::inc('val',5);
// Decrementar caché en 1
Cache::dec('val');
// Decrementar caché en 5
Cache::dec('val',5);

// Usar etiquetas de caché
Cache::tag('tag_name')->set('val','value',600);
// Eliminar caché bajo una etiqueta
Cache::tag('tag_name')->clear();
// Soporta múltiples etiquetas
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// Eliminar caché bajo múltiples etiquetas
Cache::tag(['tag1','tag2'])->clear();

// Usar diferentes almacenes de caché
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


