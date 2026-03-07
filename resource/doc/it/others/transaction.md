# Uso corretto delle transazioni

L'uso delle transazioni di database in webman è uguale a quello degli altri framework. Di seguito i punti da tenere presenti.

## Struttura del codice

La struttura del codice è la stessa degli altri framework (ad es. Laravel, think-orm simile):

```php
Db::beginTransaction();
try {
    // ..logica di business omessa...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

**Importante:** occorre usare **obbligatoriamente** `\Throwable` e **non** `\Exception`, perché nella logica di business può verificarsi un `Error`, che non eredita da `Exception`.

## Connessione al database

Quando si lavora con i modelli all'interno di una transazione, verificare se il modello ha una connessione configurata. Se il modello definisce una connessione, va specificata all'avvio della transazione; altrimenti la transazione non avrà effetto (think-orm simile). Esempio:

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // Connessione definita per il modello
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

Quando il modello definisce una connessione, va specificata per begin, commit e rollback:

```php
Db::connection('mysql')->beginTransaction();
try {
    // Logica di business
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## Individuare richieste con transazioni non confermate

A volte un bug nel codice di business lascia una transazione non confermata. Per identificare rapidamente quale metodo del controller è interessato, si può installare il componente `webman/log`. Il componente verifica automaticamente dopo ogni richiesta se ci sono transazioni non confermate e le registra nei log. La parola chiave nei log è `Uncommitted transactions`.

**Installazione di webman/log**

`composer require webman/log`

> **Nota**
> Dopo l'installazione è necessario effettuare un **restart**; reload non è sufficiente.
