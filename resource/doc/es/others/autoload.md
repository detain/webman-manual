# Carga automática

## Cargar archivos PSR-0 mediante Composer
webman sigue la especificación de carga automática `PSR-4`. Si tu proyecto necesita cargar bibliotecas compatibles con PSR-0, sigue estos pasos:

- Crea un directorio `extend` para almacenar las bibliotecas PSR-0
- Edita `composer.json` y añade lo siguiente en `autoload`:

```json
"psr-0" : {
    "": "extend/"
}
```
El resultado final será similar a:
![](../../assets/img/psr0.png)

- Ejecuta `composer dumpautoload`
- Ejecuta `php start.php restart` para reiniciar webman (nota: es necesario reiniciar para que surta efecto)

## Cargar ciertos archivos mediante Composer

- Edita `composer.json` y añade en `autoload.files` los archivos a cargar:
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- Ejecuta `composer dumpautoload`
- Ejecuta `php start.php restart` para reiniciar webman (nota: es necesario reiniciar para que surta efecto)

> **Nota**
> Los archivos configurados en `autoload.files` de composer.json se cargan antes de que arranque webman. Los archivos cargados mediante `config/autoload.php` del framework se cargan después de que arranque webman.
> Los cambios en archivos de `autoload.files` de composer.json requieren restart para aplicarse; reload no basta. Los archivos cargados mediante `config/autoload.php` del framework admiten hot-reload; los cambios se aplican tras un reload.

## Cargar ciertos archivos mediante el framework
Algunos archivos pueden no cumplir la especificación PSR y no cargarse automáticamente. Puedes cargarlos configurando `config/autoload.php`, por ejemplo:
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **Nota**
 > En `autoload.php` están configurados `support/Request.php` y `support/Response.php` porque existen archivos homónimos en `vendor/workerman/webman-framework/src/support/`. Mediante `autoload.php` se priorizan las versiones del directorio raíz del proyecto, permitiendo personalizar estos archivos sin tocar los de `vendor`. Si no necesitas personalizarlos, puedes omitir estas dos entradas.
