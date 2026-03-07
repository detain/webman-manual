# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) es un componente de base de datos basado en [top-think/think-orm](https://github.com/top-think/think-orm). Soporta pool de conexiones y funciona tanto en entornos con corutinas como sin ellas.

## Instalación

`composer require -W webman/think-orm`

Es necesario reiniciar (restart) después de la instalación (reload no tiene efecto).

## Archivo de configuración

Modifique el archivo de configuración `config/think-orm.php` según sus necesidades.

## Documentación

https://www.kancloud.cn/manual/think-orm

## Uso

```php
<?php
namespace app\controller;

use support\Request;
use support\think\Db;

class FooController
{
    public function get(Request $request)
    {
        $user = Db::table('user')->where('uid', '>', 1)->find();
        return json($user);
    }
}
```

## Creación de modelos

Los modelos de think-orm heredan de `support\think\Model`, como se muestra a continuación:

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * La tabla asociada con el modelo.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * La clave primaria asociada con la tabla.
     *
     * @var string
     */
    protected $pk = 'id';

}
```

También puede crear modelos think-orm con el siguiente comando:

```
php webman make:model nombre_tabla
```

> **Consejo**
> Este comando requiere `webman/console`. Instálelo con `composer require webman/console ^1.2.13`.

> **Nota**
> Si `make:model` detecta que el proyecto principal usa `illuminate/database`, creará archivos de modelos basados en Illuminate en lugar de think-orm. En ese caso, añada el parámetro `tp` para forzar la generación: `php webman make:model nombre_tabla tp` (actualice `webman/console` si no funciona).
