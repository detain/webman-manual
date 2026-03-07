# Datenbank

Da die meisten Plugins [webman-admin](https://www.workerman.net/plugin/82) installieren, wird empfohlen, die Datenbankkonfiguration von `webman-admin` direkt wiederzuverwenden.

Wenn die Model-Basisklasse `plugin\admin\app\model\Base` verwendet wird, kommt automatisch die Datenbankkonfiguration von webman-admin zum Einsatz.
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

Sie können die Datenbank von webman-admin auch über `plugin.admin.mysql` ansprechen, z. B.:

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## Eigene Datenbank verwenden

Plugins können auch eine eigene Datenbank nutzen. Beispielinhalt von `plugin/foo/config/database.php`:

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql ist der Verbindungsname
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'Datenbank',
            'username'    => 'Benutzername',
            'password'    => 'Passwort',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin ist der Verbindungsname
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'Datenbank',
            'username'    => 'Benutzername',
            'password'    => 'Passwort',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

Die Referenz erfolgt über `Db::connection('plugin.{Plugin}.{Verbindungsname}');`, z. B.:

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

Um die Datenbank des Hauptprojekts zu verwenden, rufen Sie sie einfach direkt auf:

```php
use support\Db;
Db::table('user')->first();
// Angenommen, das Hauptprojekt hat ebenfalls eine admin-Verbindung konfiguriert
Db::connection('admin')->table('admin')->first();
```

#### Datenbank für das Model konfigurieren

Sie können eine Base-Klasse für das Model anlegen und mit `$connection` die eigene Datenbankverbindung des Plugins festlegen:

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

Damit verwenden alle Models im Plugin, die von Base erben, automatisch die eigene Datenbank des Plugins.

## Automatischer Datenbankimport

Mit `php webman app-plugin:create foo` wird das foo-Plugin erstellt, inklusive `plugin/foo/api/Install.php` und `plugin/foo/install.sql`.

> **Hinweis**
> Wenn die Datei install.sql nicht erzeugt wird, versuchen Sie ein Update von `webman/console`: `composer require webman/console ^1.3.6`

#### Datenbank bei Plugin-Installation importieren

Beim Installieren eines Plugins wird die `install`-Methode in Install.php ausgeführt. Sie führt die SQL-Anweisungen in `install.sql` aus und importiert so die Datenbanktabellen.

Der Inhalt von `install.sql` umfasst die Tabellenerstellung und historische Schemaänderungen. Jede Anweisung muss mit `;` enden, z. B.:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Primärschlüssel',
  `order_id` varchar(50) NOT NULL COMMENT 'Bestell-ID',
  `user_id` int NOT NULL COMMENT 'Benutzer-ID',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Zu zahlender Betrag',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Bestellungen';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Primärschlüssel',
  `name` varchar(50) NOT NULL COMMENT 'Name',
  `price` int NOT NULL COMMENT 'Preis',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Ware';
```

**Datenbankverbindung ändern**

Standardmäßig wird `install.sql` in die Datenbank von webman-admin importiert. Für eine andere Datenbank passen Sie die Eigenschaft `$connection` in `Install.php` an:

```php
<?php

class Install
{
    // Eigene Datenbank des Plugins angeben
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**Test**

Führen Sie `php webman app-plugin:install foo` aus, um das Plugin zu installieren. Anschließend prüfen Sie die Datenbank – die Tabellen `foo_orders` und `foo_goods` sollten erstellt sein.

#### Tabellenstruktur beim Plugin-Upgrade ändern

Wenn ein Plugin-Upgrade neue Tabellen oder Schemaänderungen erfordert, fügen Sie die entsprechenden SQL-Anweisungen am Ende von `install.sql` an. Jede Anweisung muss mit `;` enden. Beispiel: Tabelle `foo_user` und Spalte `status` in `foo_orders` hinzufügen:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Primärschlüssel',
  `order_id` varchar(50) NOT NULL COMMENT 'Bestell-ID',
  `user_id` int NOT NULL COMMENT 'Benutzer-ID',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Zu zahlender Betrag',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Bestellungen';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Primärschlüssel',
 `name` varchar(50) NOT NULL COMMENT 'Name',
 `price` int NOT NULL COMMENT 'Preis',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Ware';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Primärschlüssel',
 `name` varchar(50) NOT NULL COMMENT 'Name'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Benutzer';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT 'Status';
```

Beim Upgrade führt die `update`-Methode in Install.php die Anweisungen aus `install.sql` aus. Neue Anweisungen werden ausgeführt, bereits angewendete werden übersprungen, sodass die Datenbankänderungen beim Upgrade korrekt übernommen werden.

#### Datenbank beim Deinstallieren des Plugins löschen

Beim Deinstallieren eines Plugins wird die `uninstall`-Methode in Install.php aufgerufen. Sie ermittelt automatisch die CREATE TABLE-Anweisungen in `install.sql` und löscht die zugehörigen Tabellen.

Wenn Sie nur Ihr eigenes `uninstall.sql` ausführen möchten und die automatische Tabellenlöschung vermeiden wollen, erstellen Sie `plugin/{Pluginname}/uninstall.sql`. Die `uninstall`-Methode führt dann nur die Anweisungen aus dieser Datei aus.
