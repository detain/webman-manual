# Database

Poiché la maggior parte dei plugin installa [webman-admin](https://www.workerman.net/plugin/82), si consiglia di riutilizzare direttamente la configurazione del database di `webman-admin`.

I modelli la cui classe base è `plugin\admin\app\model\Base` utilizzeranno automaticamente la configurazione del database di webman-admin.
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

È possibile accedere al database di webman-admin anche tramite `plugin.admin.mysql`, ad esempio:

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## Utilizzare il proprio database

I plugin possono anche scegliere di utilizzare un database proprio. Ad esempio, il contenuto di `plugin/foo/config/database.php`:

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql è il nome della connessione
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'database',
            'username'    => 'username',
            'password'    => 'password',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin è il nome della connessione
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'database',
            'username'    => 'username',
            'password'    => 'password',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

Il formato di riferimento è `Db::connection('plugin.{plugin}.{nome_connessione}');`, ad esempio:

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

Per utilizzare il database del progetto principale, invocatelo direttamente:

```php
use support\Db;
Db::table('user')->first();
// Supponendo che il progetto principale abbia anche una connessione admin configurata
Db::connection('admin')->table('admin')->first();
```

#### Configurare il database per il Model

È possibile creare una classe Base per il Model e impostare la proprietà `$connection` per utilizzare la connessione al database propria del plugin:

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

In questo modo tutti i modelli del plugin che ereditano da Base utilizzeranno automaticamente il database proprio del plugin.

## Importazione automatica del database

L'esecuzione di `php webman app-plugin:create foo` crea il plugin foo, incluse `plugin/foo/api/Install.php` e `plugin/foo/install.sql`.

> **Suggerimento**
> Se il file install.sql non viene generato, provare ad aggiornare `webman/console` con: `composer require webman/console ^1.3.6`

#### Importare il database durante l'installazione del plugin

Quando si installa un plugin, viene eseguito il metodo `install` in Install.php, che esegue automaticamente le istruzioni SQL in `install.sql`, importando così le tabelle del database.

Il contenuto di `install.sql` deve includere la creazione delle tabelle e le modifiche storiche dello schema. Ogni istruzione deve terminare con `;`, ad esempio:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Chiave primaria',
  `order_id` varchar(50) NOT NULL COMMENT 'ID ordine',
  `user_id` int NOT NULL COMMENT 'ID utente',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Importo da pagare',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Ordini';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Chiave primaria',
  `name` varchar(50) NOT NULL COMMENT 'Nome',
  `price` int NOT NULL COMMENT 'Prezzo',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Prodotti';
```

**Modificare la connessione al database**

Per impostazione predefinita, `install.sql` viene importato nel database di webman-admin. Per importare in un database diverso, modificare la proprietà `$connection` in `Install.php`:

```php
<?php

class Install
{
    // Specificare il database proprio del plugin
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**Test**

Eseguire `php webman app-plugin:install foo` per installare il plugin. Poi verificare il database: le tabelle `foo_orders` e `foo_goods` dovrebbero essere state create.

#### Modificare la struttura delle tabelle durante l'aggiornamento del plugin

Quando un aggiornamento del plugin richiede nuove tabelle o modifiche allo schema, aggiungere le istruzioni SQL corrispondenti alla fine di `install.sql`. Ogni istruzione deve terminare con `;`. Ad esempio, aggiungere la tabella `foo_user` e la colonna `status` a `foo_orders`:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Chiave primaria',
  `order_id` varchar(50) NOT NULL COMMENT 'ID ordine',
  `user_id` int NOT NULL COMMENT 'ID utente',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Importo da pagare',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Ordini';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Chiave primaria',
 `name` varchar(50) NOT NULL COMMENT 'Nome',
 `price` int NOT NULL COMMENT 'Prezzo',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Prodotti';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Chiave primaria',
 `name` varchar(50) NOT NULL COMMENT 'Nome'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Utente';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT 'Stato';
```

Durante l'aggiornamento, il metodo `update` in Install.php esegue le istruzioni di `install.sql`. Le nuove vengono eseguite; quelle già applicate vengono saltate, applicando così correttamente le modifiche al database durante gli aggiornamenti.

#### Eliminare il database alla disinstallazione del plugin

Alla disinstallazione di un plugin viene chiamato il metodo `uninstall` in Install.php. Questo analizza automaticamente le istruzioni CREATE TABLE in `install.sql` ed elimina quelle tabelle.

Se si desidera eseguire solo il proprio `uninstall.sql` invece dell'eliminazione automatica delle tabelle, creare `plugin/{nome_plugin}/uninstall.sql`. In questo caso il metodo `uninstall` eseguirà solo le istruzioni di quel file.
