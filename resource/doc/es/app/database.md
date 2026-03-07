# Base de datos

Dado que la mayoría de los plugins instalan [webman-admin](https://www.workerman.net/plugin/82), se recomienda reutilizar directamente la configuración de la base de datos de `webman-admin`.

Los modelos cuya clase base es `plugin\admin\app\model\Base` utilizarán automáticamente la configuración de la base de datos de webman-admin.
```php
<?php

namespace plugin\foo\app\model;

use plugin\admin\app\model\Base;

class Orders extends Base
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'foo_orders';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $primaryKey = 'id';
    
}
```

También puede acceder a la base de datos de webman-admin mediante `plugin.admin.mysql`, por ejemplo:

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## Usar tu propia base de datos

Los plugins también pueden elegir usar su propia base de datos. Por ejemplo, el contenido de `plugin/foo/config/database.php`:

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql es el nombre de la conexión
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'base_de_datos',
            'username'    => 'usuario',
            'password'    => 'contraseña',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin es el nombre de la conexión
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'base_de_datos',
            'username'    => 'usuario',
            'password'    => 'contraseña',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

El formato de referencia es `Db::connection('plugin.{plugin}.{nombre_conexión}');`, por ejemplo:

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

Para usar la base de datos del proyecto principal, invóquela directamente:

```php
use support\Db;
Db::table('user')->first();
// Suponiendo que el proyecto principal también tiene configurada una conexión admin
Db::connection('admin')->table('admin')->first();
```

#### Configurar la base de datos para el Model

Puede crear una clase Base para el Model y establecer la propiedad `$connection` para usar la conexión a la base de datos propia del plugin:

```php
<?php

namespace plugin\foo\app\model;

use DateTimeInterface;
use support\Model;

class Base extends Model
{
    /**
     * @var string
     */
    protected $connection = 'plugin.foo.mysql';

}
```

Todos los modelos del plugin que hereden de Base utilizarán así automáticamente la base de datos propia del plugin.

## Importación automática de la base de datos

Al ejecutar `php webman app-plugin:create foo` se crea el plugin foo, incluyendo `plugin/foo/api/Install.php` y `plugin/foo/install.sql`.

> **Sugerencia**
> Si no se genera el archivo install.sql, intente actualizar `webman/console` con: `composer require webman/console ^1.3.6`

#### Importar la base de datos al instalar el plugin

Al instalar un plugin, se ejecuta el método `install` en Install.php, que lanza automáticamente las sentencias SQL de `install.sql`, importando así las tablas de la base de datos.

El contenido de `install.sql` debe incluir la creación de tablas y los cambios históricos del esquema. Cada sentencia debe terminar con `;`, por ejemplo:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Clave primaria',
  `order_id` varchar(50) NOT NULL COMMENT 'ID pedido',
  `user_id` int NOT NULL COMMENT 'ID usuario',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Importe a pagar',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Pedidos';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Clave primaria',
  `name` varchar(50) NOT NULL COMMENT 'Nombre',
  `price` int NOT NULL COMMENT 'Precio',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Productos';
```

**Cambiar la conexión a la base de datos**

Por defecto, `install.sql` se importa en la base de datos de webman-admin. Para importar en otra base de datos, modifique la propiedad `$connection` en `Install.php`:

```php
<?php

class Install
{
    // Especificar la base de datos propia del plugin
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**Prueba**

Ejecute `php webman app-plugin:install foo` para instalar el plugin. Después, compruebe la base de datos: las tablas `foo_orders` y `foo_goods` deberían estar creadas.

#### Cambiar la estructura de tablas durante la actualización del plugin

Cuando una actualización del plugin requiere nuevas tablas o cambios en el esquema, añada las sentencias SQL correspondientes al final de `install.sql`. Cada sentencia debe terminar con `;`. Por ejemplo, añadir la tabla `foo_user` y la columna `status` a `foo_orders`:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Clave primaria',
  `order_id` varchar(50) NOT NULL COMMENT 'ID pedido',
  `user_id` int NOT NULL COMMENT 'ID usuario',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Importe a pagar',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Pedidos';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Clave primaria',
 `name` varchar(50) NOT NULL COMMENT 'Nombre',
 `price` int NOT NULL COMMENT 'Precio',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Productos';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Clave primaria',
 `name` varchar(50) NOT NULL COMMENT 'Nombre'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Usuario';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT 'Estado';
```

Durante la actualización, el método `update` en Install.php ejecuta las sentencias de `install.sql`. Las nuevas se ejecutan; las ya aplicadas se omiten, aplicando así correctamente los cambios en la base de datos durante las actualizaciones.

#### Eliminar la base de datos al desinstalar el plugin

Al desinstalar un plugin, se llama al método `uninstall` en Install.php. Este analiza automáticamente las sentencias CREATE TABLE en `install.sql` y elimina esas tablas.

Si solo quiere ejecutar su propio `uninstall.sql` en lugar de la eliminación automática de tablas, cree `plugin/{nombre_plugin}/uninstall.sql`. En ese caso, el método `uninstall` ejecutará únicamente las sentencias de ese archivo.
