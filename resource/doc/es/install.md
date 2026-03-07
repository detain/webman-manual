# Cómo instalar webman

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux: Instalar PHP + Composer (omitir si ya está configurado)
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **Nota**
> El comando anterior aplica a Linux y macOS. Los usuarios de Windows deben instalar PHP por separado.

También puede descargar manualmente la [versión estática de PHP](https://www.workerman.net/download) proporcionada por webman y extraerla para usarla.

## 1. Crear proyecto

```php
composer create-project workerman/webman:~2.0
```

> **Consejo**
> Si hay errores, puede que esté usando un espejo de Composer defectuoso. Ejecute `composer config -g --unset repos.packagist` para quitar el proxy.

## 2. Ejecutar

Entre en el directorio de webman

#### Usuarios de Windows
Haga doble clic en `windows.bat` o ejecute `php windows.php` para iniciar

> **Consejo**
> Si hay errores, algunas funciones pueden estar deshabilitadas. Consulte [Verificación de funciones deshabilitadas](others/disable-function-check.md) para habilitarlas.

#### Usuarios de Linux
**Modo depuración** (para desarrollo: la salida aparece en la terminal; el servicio se detiene al cerrar la terminal)

```php
php start.php start
```

**Modo daemon** (para producción: no hay salida en la terminal; el servicio sigue ejecutándose tras cerrar la terminal)

```php
php start.php start -d
```

#### Usuarios de Docker

Iniciar todos los servicios y adjuntar a la consola
```php
docker-compose up
```

Ejecutar servicios en segundo plano
```php
docker-compose up -d
```

> **Consejo**
> Si hay errores, algunas funciones pueden estar deshabilitadas. Consulte [Verificación de funciones deshabilitadas](others/disable-function-check.md) para habilitarlas.

## 3. Acceder

Abra `http://dirección-ip:8787` en el navegador.
