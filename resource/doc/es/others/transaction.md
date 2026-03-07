# Uso correcto de transacciones

El uso de transacciones de base de datos en webman es igual que en otros frameworks. Aquí se enumeran los puntos que conviene tener en cuenta.

## Estructura del código

La estructura del código es la misma que en otros frameworks (por ejemplo Laravel, think-orm es similar):

```php
Db::beginTransaction();
try {
    // ..lógica de negocio omitida...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

**Importante:** debe usar **obligatoriamente** `\Throwable` y **no** `\Exception`, ya que la lógica de negocio puede provocar un `Error`, que no hereda de `Exception`.

## Conexión a la base de datos

Al operar con modelos dentro de una transacción, hay que prestar atención a si el modelo tiene configurada una conexión. Si el modelo define una conexión, debe especificarla al iniciar la transacción; de lo contrario la transacción no tendrá efecto (think-orm es similar). Por ejemplo:

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // Conexión definida para el modelo
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

Cuando el modelo define una conexión, debe especificarla para begin, commit y rollback:

```php
Db::connection('mysql')->beginTransaction();
try {
    // Lógica de negocio
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## Localizar solicitudes con transacciones no confirmadas

A veces, un fallo en el código de negocio deja una transacción sin confirmar. Para localizar rápidamente qué método del controlador la provoca, puede instalar el componente `webman/log`. Este componente comprueba tras cada solicitud si hay transacciones sin confirmar y las registra en los logs. La palabra clave en el log es `Uncommitted transactions`.

**Instalación de webman/log**

`composer require webman/log`

> **Nota**
> Tras la instalación es necesario **reiniciar** (restart); reload no surte efecto.
