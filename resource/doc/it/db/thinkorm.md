# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) è un componente di database basato su [top-think/think-orm](https://github.com/top-think/think-orm). Supporta il connection pooling e funziona sia in ambienti coroutine che non-coroutine.

## Installazione

`composer require -W webman/think-orm`

È necessario riavviare (restart) dopo l'installazione (reload non ha effetto).

## File di configurazione

Modifica il file di configurazione `config/think-orm.php` in base alle tue esigenze.

## Documentazione

https://www.kancloud.cn/manual/think-orm

## Utilizzo

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

## Creazione di modelli

I modelli think-orm estendono `support\think\Model`, come mostrato di seguito:

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * La tabella associata al modello.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * La chiave primaria associata alla tabella.
     *
     * @var string
     */
    protected $pk = 'id';

}
```

Puoi anche creare modelli think-orm con il seguente comando:

```
php webman make:model nome_tabella
```

> **Suggerimento**
> Questo comando richiede `webman/console`. Installalo con `composer require webman/console ^1.2.13`.

> **Nota**
> Se `make:model` rileva che il progetto principale utilizza `illuminate/database`, creerà file di modello basati su Illuminate invece che su think-orm. In quel caso, aggiungi il parametro `tp`: `php webman make:model nome_tabella tp` (aggiorna `webman/console` se non funziona).
