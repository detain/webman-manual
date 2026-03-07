# Base de données

Comme la plupart des plugins installent [webman-admin](https://www.workerman.net/plugin/82), il est recommandé de réutiliser directement la configuration de la base de données de `webman-admin`.

Les modèles dont la classe de base est `plugin\admin\app\model\Base` utilisent automatiquement la configuration de la base de données de webman-admin.
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

Vous pouvez également accéder à la base de données de webman-admin via `plugin.admin.mysql`, par exemple :

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## Utiliser sa propre base de données

Les plugins peuvent également choisir d'utiliser leur propre base de données. Par exemple, le contenu de `plugin/foo/config/database.php` :

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql est le nom de la connexion
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'base_de_données',
            'username'    => 'nom_utilisateur',
            'password'    => 'mot_de_passe',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin est le nom de la connexion
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'base_de_données',
            'username'    => 'nom_utilisateur',
            'password'    => 'mot_de_passe',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

La syntaxe de référence est `Db::connection('plugin.{plugin}.{nom_connexion}');`, par exemple :

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

Pour utiliser la base de données du projet principal, appelez-la directement :

```php
use support\Db;
Db::table('user')->first();
// En supposant que le projet principal a aussi une connexion admin configurée
Db::connection('admin')->table('admin')->first();
```

#### Configurer la base de données pour le Model

Vous pouvez créer une classe Base pour le Model et définir la propriété `$connection` pour utiliser la connexion à la base de données propre au plugin :

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

Tous les modèles du plugin qui héritent de Base utiliseront ainsi automatiquement la base de données propre au plugin.

## Import automatique de la base de données

L'exécution de `php webman app-plugin:create foo` crée le plugin foo, incluant `plugin/foo/api/Install.php` et `plugin/foo/install.sql`.

> **Conseil**
> Si le fichier install.sql n'est pas généré, essayez de mettre à jour `webman/console` : `composer require webman/console ^1.3.6`

#### Import de la base de données lors de l'installation du plugin

Lors de l'installation d'un plugin, la méthode `install` dans Install.php est exécutée et lance automatiquement les instructions SQL de `install.sql`, important ainsi les tables de la base de données.

Le contenu de `install.sql` doit inclure la création des tables et les modifications historiques du schéma. Chaque instruction doit se terminer par `;`, par exemple :

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Clé primaire',
  `order_id` varchar(50) NOT NULL COMMENT 'ID commande',
  `user_id` int NOT NULL COMMENT 'ID utilisateur',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Montant à payer',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Commandes';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Clé primaire',
  `name` varchar(50) NOT NULL COMMENT 'Nom',
  `price` int NOT NULL COMMENT 'Prix',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Produits';
```

**Modifier la connexion à la base de données**

Par défaut, `install.sql` est importé dans la base de données de webman-admin. Pour importer dans une autre base de données, modifiez la propriété `$connection` dans `Install.php` :

```php
<?php

class Install
{
    // Spécifier la base de données propre au plugin
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**Test**

Exécutez `php webman app-plugin:install foo` pour installer le plugin, puis vérifiez la base de données : les tables `foo_orders` et `foo_goods` devraient être créées.

#### Modifier la structure des tables lors de la mise à jour du plugin

Lorsqu'une mise à jour de plugin nécessite de nouvelles tables ou des modifications du schéma, ajoutez les instructions SQL correspondantes à la fin de `install.sql`. Chaque instruction doit se terminer par `;`. Par exemple, ajouter une table `foo_user` et une colonne `status` dans `foo_orders` :

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Clé primaire',
  `order_id` varchar(50) NOT NULL COMMENT 'ID commande',
  `user_id` int NOT NULL COMMENT 'ID utilisateur',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Montant à payer',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Commandes';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Clé primaire',
 `name` varchar(50) NOT NULL COMMENT 'Nom',
 `price` int NOT NULL COMMENT 'Prix',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Produits';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Clé primaire',
 `name` varchar(50) NOT NULL COMMENT 'Nom'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Utilisateur';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT 'Statut';
```

Lors de la mise à jour, la méthode `update` dans Install.php exécute les instructions de `install.sql`. Les nouvelles instructions sont exécutées ; les anciennes déjà appliquées sont ignorées, de sorte que les modifications de la base de données sont correctement appliquées lors des mises à jour.

#### Suppression de la base de données lors de la désinstallation du plugin

Lors de la désinstallation d'un plugin, la méthode `uninstall` dans Install.php est appelée. Elle analyse automatiquement les instructions CREATE TABLE dans `install.sql` et supprime ces tables.

Si vous souhaitez exécuter uniquement votre propre `uninstall.sql` au lieu de la suppression automatique des tables, créez `plugin/{nom_plugin}/uninstall.sql`. La méthode `uninstall` n'exécutera alors que les instructions de ce fichier.
