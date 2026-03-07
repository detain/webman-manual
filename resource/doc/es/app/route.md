# Archivo de configuración de rutas
El archivo de configuración de rutas de los plugins se encuentra en `plugin/nombre_del_plugin/config/route.php`.

## Ruta predeterminada
Todas las rutas de URL de los plugins de aplicación comienzan con `/app`, por ejemplo la URL de `plugin\foo\app\controller\UserController` es `http://127.0.0.1:8787/app/foo/user`.

## Deshabilitar ruta predeterminada
Para deshabilitar la ruta predeterminada de un plugin de aplicación, añada lo siguiente en la configuración de rutas:
```php
Route::disableDefaultRoute('foo');
```

## Manejar devolución de llamada 404
Para definir un fallback para un plugin de aplicación, hay que pasar el nombre del plugin como segundo parámetro. Ejemplo:
```php
Route::fallback(function(){
    return redirect('/');
}, 'foo');
```
