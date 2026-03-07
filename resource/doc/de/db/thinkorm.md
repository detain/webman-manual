# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) ist eine Datenbankkomponente basierend auf [top-think/think-orm](https://github.com/top-think/think-orm). Sie unterstützt Connection Pooling und funktioniert sowohl in Coroutine- als auch in Nicht-Coroutine-Umgebungen.

## Installation

`composer require -W webman/think-orm`

Nach der Installation ist ein Neustart (restart) erforderlich (reload hat keine Wirkung).

## Konfigurationsdatei

Passen Sie die Konfigurationsdatei `config/think-orm.php` entsprechend Ihren Anforderungen an.

## Dokumentation

https://www.kancloud.cn/manual/think-orm

## Verwendung

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

## Modell erstellen

Think-orm-Modelle erben von `support\think\Model`, wie folgt:

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * Die mit dem Modell verknüpfte Tabelle.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * Der Primärschlüssel der Tabelle.
     *
     * @var string
     */
    protected $pk = 'id';

}
```

Sie können auch Think-orm-Modelle mit folgendem Befehl erstellen:

```
php webman make:model tabellenname
```

> **Tipp**
> Dieser Befehl erfordert `webman/console`. Installieren Sie es mit `composer require webman/console ^1.2.13`.

> **Hinweis**
> Wenn `make:model` erkennt, dass das Hauptprojekt `illuminate/database` verwendet, werden Illuminate-basierte Modell-Dateien erstellt statt think-orm. In diesem Fall fügen Sie den Parameter `tp` hinzu: `php webman make:model tabellenname tp` (aktualisieren Sie `webman/console`, falls es nicht funktioniert).
