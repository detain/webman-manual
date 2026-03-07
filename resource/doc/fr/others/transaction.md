# Utilisation correcte des transactions

L'utilisation des transactions en base de données dans webman est identique aux autres frameworks. Voici les points importants à noter.

## Structure du code

La structure du code est la même que dans les autres frameworks (par exemple Laravel, think-orm similaire) :

```php
Db::beginTransaction();
try {
    // ..logique métier omise...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

**Important :** vous devez **obligatoirement** utiliser `\Throwable` et **ne pas** utiliser `\Exception`, car la logique métier peut déclencher une `Error`, qui n'hérite pas de `Exception`.

## Connexion à la base de données

Lors de l'utilisation de modèles dans une transaction, veillez à vérifier si le modèle a une connexion configurée. Si le modèle définit une connexion, vous devez la préciser au démarrage de la transaction ; sinon la transaction ne sera pas effective (think-orm similaire). Exemple :

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // Connexion définie pour le modèle
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

Lorsque le modèle définit une connexion, vous devez la préciser pour begin, commit et rollback :

```php
Db::connection('mysql')->beginTransaction();
try {
    // Logique métier
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## Trouver les requêtes avec transactions non validées

Parfois, un bug dans le code métier laisse une transaction non validée. Pour identifier rapidement quelle méthode de contrôleur est concernée, vous pouvez installer le composant `webman/log`. Il vérifie automatiquement après chaque requête s'il reste des transactions non validées et les enregistre dans les logs. Le mot-clé des logs est `Uncommitted transactions`.

**Installation de webman/log**

`composer require webman/log`

> **Note**
> Un **redémarrage** (restart) est nécessaire après l'installation ; reload ne suffit pas.
