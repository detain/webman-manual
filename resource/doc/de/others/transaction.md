# Korrekte Verwendung von Datenbanktransaktionen

Die Verwendung von Datenbanktransaktionen in webman entspricht der in anderen Frameworks. Hier die wichtigsten Punkte:

## Codestruktur

Die Codestruktur entspricht der in anderen Frameworks (z. B. Laravel, think-orm ähnlich):

```php
Db::beginTransaction();
try {
    // ..Geschäftslogik ausgelassen...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

**Wichtig:** Sie müssen `\Throwable` verwenden und dürfen nicht `\Exception` nutzen, da in der Geschäftslogik `Error` ausgelöst werden kann, der nicht von `Exception` erbt.

## Datenbankverbindung

Bei der Arbeit mit Modellen innerhalb einer Transaktion ist zu beachten, ob das Modell eine Verbindung konfiguriert hat. Wenn das Modell eine Verbindung festlegt, muss diese beim Start der Transaktion angegeben werden; andernfalls greift die Transaktion nicht (think-orm ähnlich). Beispiel:

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // Verbindung für das Modell festgelegt
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

Wenn das Modell eine Verbindung festlegt, müssen Sie für begin, commit und rollback die Verbindung angeben:

```php
Db::connection('mysql')->beginTransaction();
try {
    // Geschäftslogik
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## Anfragen mit nicht bestätigten Transaktionen finden

Manchmal führt ein Fehler im Geschäftscode dazu, dass eine Transaktion nicht bestätigt wird. Um schnell zu ermitteln, welche Controller-Methode betroffen ist, können Sie die Komponente `webman/log` installieren. Diese prüft nach jeder Anfrage automatisch auf nicht bestätigte Transaktionen und protokolliert sie. Das Protokoll-Schlüsselwort lautet `Uncommitted transactions`.

**Installation von webman/log**

`composer require webman/log`

> **Hinweis**
> Nach der Installation muss ein Neustart (**restart**) erfolgen; **reload** reicht nicht aus.
