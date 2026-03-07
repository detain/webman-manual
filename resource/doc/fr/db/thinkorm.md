# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) est un composant de base de données basé sur [top-think/think-orm](https://github.com/top-think/think-orm). Il prend en charge le pool de connexions et fonctionne en environnement coroutine et non-coroutine.

## Installation

`composer require -W webman/think-orm`

Un redémarrage (restart) est nécessaire après l'installation (reload n'est pas effectif).

## Fichier de configuration

Modifiez le fichier de configuration `config/think-orm.php` selon vos besoins.

## Documentation

https://www.kancloud.cn/manual/think-orm

## Utilisation

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

## Création de modèles

Les modèles think-orm héritent de `support\think\Model`, comme suit :

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * La table associée au modèle.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * La clé primaire associée à la table.
     *
     * @var string
     */
    protected $pk = 'id';

}
```

Vous pouvez également créer des modèles think-orm avec la commande suivante :

```
php webman make:model nom_table
```

> **Astuce**
> Cette commande nécessite `webman/console`. Installez-le avec `composer require webman/console ^1.2.13`.

> **Remarque**
> Si `make:model` détecte que le projet principal utilise `illuminate/database`, il créera des fichiers de modèles Illuminate au lieu de think-orm. Dans ce cas, ajoutez le paramètre `tp` : `php webman make:model nom_table tp` (mettez à jour `webman/console` si cela ne fonctionne pas).
