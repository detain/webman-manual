# Flujo básico de generación y publicación de plugins

## Principio
1. Tomando como ejemplo un plugin de cross-origin: está compuesto por tres partes: el archivo del middleware de cross-origin, el archivo de configuración `middleware.php` y un `Install.php` generado automáticamente por comando.
2. Se usa un comando para empaquetar estos tres archivos y publicarlos en Composer.
3. Cuando el usuario instala el plugin de cross-origin con Composer, `Install.php` copia el archivo del middleware y la configuración a `{proyecto principal}/config/plugin` para que webman los cargue, activando así la configuración cross-origin.
4. Cuando el usuario desinstala el plugin con Composer, `Install.php` elimina los archivos del middleware y la configuración correspondientes, realizando la desinstalación automática del plugin.

## Especificaciones
1. El nombre del plugin se compone de `vendor` y `nombre del plugin`, p. ej. `webman/push`, que corresponde al nombre del paquete Composer.
2. Los archivos de configuración del plugin se colocan en `config/plugin/vendor/nombre del plugin/` (el comando de consola crea el directorio de configuración automáticamente). Si el plugin no necesita configuración, hay que eliminar el directorio creado automáticamente.
3. El directorio de configuración del plugin solo admite: `app.php` (config principal), `bootstrap.php` (arranque de procesos), `route.php` (rutas), `middleware.php` (middleware), `process.php` (procesos personalizados), `database.php` (base de datos), `redis.php` (Redis), `thinkorm.php` (thinkorm). Estos se reconocen automáticamente por webman.
4. Acceso a la configuración: `config('plugin.vendor.nombre del plugin.archivo de config.clave');`, p. ej. `config('plugin.webman.push.app.app_key')`.
5. Si el plugin tiene su propia configuración de BD: para `illuminate/database` usar `Db::connection('plugin.vendor.nombre del plugin.conexión')`, para `thinkorm` usar `Db::connect('plugin.vendor.nombre del plugin.conexión')`.
6. Si un plugin coloca archivos de negocio en `app/`, debe evitar conflictos con el proyecto principal y otros plugins.
7. Los plugins deben evitar copiar archivos o directorios al proyecto principal. P. ej., en el plugin cross-origin solo se copia la configuración; los archivos del middleware se mantienen en `vendor/webman/cros/src`.
8. Se recomienda PascalCase para los namespaces de los plugins, p. ej. `Webman/Console`.

## Ejemplo

**Instalar la línea de comandos `webman/console`**

`composer require webman/console`

### Crear un plugin

Supongamos que el plugin a crear se llama `foo/admin` (es el nombre del proyecto Composer y debe ir en minúsculas). Ejecutar:

`php webman plugin:create --name=foo/admin`

Se crearán `vendor/foo/admin` para los archivos del plugin y `config/plugin/foo/admin` para la configuración.

> Nota
> `config/plugin/foo/admin` admite: `app.php`, `bootstrap.php`, `route.php`, `middleware.php`, `process.php`, `database.php`, `redis.php`, `thinkorm.php`. Mismo formato que webman, se fusionan automáticamente.
> Acceso con prefijo `plugin`, p. ej. `config('plugin.foo.admin.app')`.


### Exportar el plugin

Una vez completado el desarrollo, ejecutar:

`php webman plugin:export --name=foo/admin`

Exportar

> Explicación
> Al exportar se copia `config/plugin/foo/admin` a `vendor/foo/admin/src` y se genera `Install.php`, que se ejecuta en la instalación y desinstalación.
> Instalación por defecto: copiar la configuración de `vendor/foo/admin/src` al `config/plugin` del proyecto.
> Desinstalación por defecto: eliminar los archivos de configuración del plugin en `config/plugin` del proyecto.
> Se puede modificar `Install.php` para añadir lógica personalizada en instalación y desinstalación.

### Enviar el plugin
* Supongamos que ya tiene una cuenta en [GitHub](https://github.com) y [Packagist](https://packagist.org).
* Cree un repositorio `admin` en [GitHub](https://github.com) y suba el código, p. ej. `https://github.com/tu-usuario/admin`.
* Vaya a `https://github.com/tu-usuario/admin/releases/new` y cree un release, p. ej. `v1.0.0`.
* En [Packagist](https://packagist.org) haga clic en `Submit` y envíe `https://github.com/tu-usuario/admin` para publicar el plugin.

> **Consejo**
> Si Packagist indica conflicto de nombre, use otro vendor, p. ej. cambiar `foo/admin` por `myfoo/admin`.

En actualizaciones: suba el código a GitHub, cree un nuevo release en `https://github.com/tu-usuario/admin/releases/new` y en `https://packagist.org/packages/foo/admin` haga clic en `Update`.

## Añadir comandos al plugin
Algunos plugins necesitan comandos personalizados. P. ej., al instalar `webman/redis-queue`, el proyecto obtiene el comando `redis-queue:consumer`. Ejecutando `php webman redis-queue:consumer send-mail` se genera rápidamente una clase consumer `SendMail.php`, lo que facilita el desarrollo.

Para añadir el comando `foo-admin:add` al plugin `foo/admin`:

### Crear un comando

**Crear el archivo `vendor/foo/admin/src/FooAdminAddCommand.php`**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = 'Descripción del comando';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **Nota**
> Para evitar conflictos de comandos entre plugins, usar el formato `vendor-plugin:comando`. Por ejemplo, todos los comandos de `foo/admin` deben usar el prefijo `foo-admin:`, p. ej. `foo-admin:add`.

### Añadir configuración
**Crear `config/plugin/foo/admin/command.php`**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // Añadir más si hace falta...
];
```

> **Consejo**
> `command.php` registra los comandos personalizados del plugin. Cada entrada es una clase de comando. `webman/console` los carga automáticamente. Ver [Comandos de consola](console.md).

### Ejecutar la exportación
Ejecutar `php webman plugin:export --name=foo/admin` para exportar y publicar en Packagist. Después de instalar `foo/admin`, estará disponible el comando `foo-admin:add`. Al ejecutar `php webman foo-admin:add jerry` se imprimirá `Admin add jerry`.
