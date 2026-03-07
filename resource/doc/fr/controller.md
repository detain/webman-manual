# Contrôleur

Créez le fichier de contrôleur `app/controller/FooController.php`.

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

Lors de l'accès à `http://127.0.0.1:8787/foo`, la page renvoie `hello index`.

Lors de l'accès à `http://127.0.0.1:8787/foo/hello`, la page renvoie `hello webman`.

Vous pouvez modifier les règles de routage via la configuration des routes, voir [Routes](route.md).

> **Conseil**
> En cas d'erreur 404, ouvrez `config/app.php`, définissez `controller_suffix` sur `Controller` et redémarrez.

## Suffixe du contrôleur
À partir de la version 1.3, webman permet de définir un suffixe de contrôleur dans `config/app.php`. Si `controller_suffix` est défini sur une chaîne vide `''`, le contrôleur ressemblera à ceci :

`app\controller\Foo.php`.

```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

Il est fortement recommandé de définir le suffixe du contrôleur sur `Controller` afin d'éviter les conflits avec les noms de classes de modèles et d'améliorer la sécurité.

## Explication
- Le framework transmet automatiquement l'objet `support\Request` au contrôleur, permettant d'obtenir les données d'entrée utilisateur (get, post, header, cookie, etc.), voir [Requête](request.md).
- Le contrôleur peut renvoyer des nombres, des chaînes ou des objets `support\Response`, mais pas d'autres types de données.
- Les objets `support\Response` peuvent être créés avec les fonctions d'aide `response()`, `json()`, `xml()`, `jsonp()`, `redirect()`, etc.

## Liaison des paramètres du contrôleur

#### Exemple
webman supporte la liaison automatique des paramètres de requête aux paramètres des méthodes du contrôleur. Par exemple :

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

Vous pouvez transmettre les valeurs de `name` et `age` via `GET` ou `POST`, ou via les paramètres de route. Par exemple :

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

La priorité est : `paramètres de route` > `GET` > `POST`.

#### Valeurs par défaut

Si vous accédez à `/user/create?name=tom`, vous obtiendrez l'erreur suivante :

```html
Missing input parameter age
```

Cela vient du fait que nous n'avons pas transmis le paramètre `age`. Vous pouvez définir une valeur par défaut pour résoudre ce problème. Par exemple :

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age = 18): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

#### Types de paramètres
Si vous accédez à `/user/create?name=tom&age=not_int`, vous obtiendrez l'erreur suivante :

> **Conseil**
> Pour faciliter les tests, nous passons les paramètres directement dans la barre d'adresse. En développement, les paramètres devraient être transmis via `POST`.

```html
Input age must be of type int, string given
```

Les données reçues sont converties selon le type. En cas d'échec de conversion, une exception `support\exception\InputTypeException` est levée. Comme `age` ne peut pas être converti en `int`, cette erreur apparaît.

#### Messages d'erreur personnalisés
Vous pouvez personnaliser des messages comme `Missing input parameter age` et `Input age must be of type int, string given` via la traduction. Référez-vous aux commandes suivantes :

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => 'Le paramètre d\'entrée :parameter doit être de type :exceptType, le type transmis est :actualType',
    'Missing input parameter :parameter' => 'Paramètre d\'entrée :parameter manquant',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### Autres types
webman supporte les types `int`, `float`, `string`, `bool`, `array`, `object` et les `instances de classe`. Par exemple :

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age, float $balance, bool $vip, array $extension): Response
    {
        return json([
            'name' => $name,
            'age' => $age,
            'balance' => $balance,
            'vip' => $vip,
            'extension' => $extension,
        ]);
    }
}
```

En accédant à `/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar`, vous obtiendrez :

```json
{
  "name": "tom",
  "age": 18,
  "balance": 100.5,
  "vip": true,
  "extension": {
    "foo": "bar"
  }
}
```

#### Instance de classe
webman permet de transmettre des instances de classes via le typage des paramètres. Par exemple :

**app\service\Blog.php**
```php
<?php
namespace app\service;
class Blog
{
    private $title;
    private $content;
    public function __construct(string $title, string $content)
    {
        $this->title = $title;
        $this->content = $content;
    }
    public function get()
    {
        return [
            'title' => $this->title,
            'content' => $this->content,
        ];
    }
}
```

**app\controller\BlogController.php**
```php
<?php
namespace app\controller;
use app\service\Blog;
use support\Response;

class BlogController
{
    public function create(Blog $blog): Response
    {
        return json($blog->get());
    }
}
```

En accédant à `/blog/create?blog[title]=hello&blog[content]=world`, vous obtiendrez :

```json
{
  "title": "hello",
  "content": "world"
}
```

#### Instance de modèle

**app\model\User.php**
```php
<?php
namespace app\model;
use support\Model;
class User extends Model
{
    protected $connection = 'mysql';
    protected $table = 'user';
    protected $primaryKey = 'id';
    public $timestamps = false;
    // Définir ici les champs remplissables pour éviter les champs non sécurisés depuis le frontend
    protected $fillable = ['name', 'age'];
}
```

**app\controller\UserController.php**
```php
<?php
namespace app\controller;
use app\model\User;
class UserController
{
    public function create(User $user): int
    {
        $user->save();
        return $user->id;
    }
}
```

En accédant à `/user/create?user[name]=tom&user[age]=18`, vous obtiendrez un résultat similaire à :

```json
1
```

## Cycle de vie du contrôleur

Lorsque `controller_reuse` dans `config/app.php` est `false`, chaque requête initialise une fois une instance du contrôleur, qui est détruite après la fin de la requête. C'est le comportement des frameworks traditionnels.

Lorsque `controller_reuse` dans `config/app.php` est `true`, toutes les requêtes réutilisent la même instance du contrôleur, qui reste en mémoire une fois créée.

> **Attention**
> Quand la réutilisation est activée, les requêtes ne doivent pas modifier les propriétés du contrôleur, car ces modifications affecteraient les requêtes suivantes. Par exemple :

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    protected $model;
    
    public function update(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->update();
        return response('ok');
    }
    
    public function delete(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->delete();
        return response('ok');
    }
    
    protected function getModel($id)
    {
        // Cette méthode conserve le modèle après la première requête update?id=1
        // Une requête delete?id=2 supprimera les données de l'id 1
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **Conseil**
> Retourner des données dans le constructeur `__construct()` du contrôleur n'a aucun effet. Par exemple :

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // Le return dans le constructeur n'a aucun effet ; le navigateur ne recevra pas cette réponse
        return response('hello'); 
    }
}
```

## Différence entre non-réutilisation et réutilisation du contrôleur

#### Sans réutilisation
Chaque requête crée une nouvelle instance du contrôleur, libérée après la requête. Comportement traditionnel, familier à la plupart des développeurs. Performance légèrement inférieure (environ 10 % en benchmark helloworld, souvent négligeable en pratique).

#### Avec réutilisation
Une instance est créée une fois par processus et conservée. Les requêtes suivantes la réutilisent. Meilleure performance, mais moins courant pour beaucoup de développeurs.

#### La réutilisation n'est pas possible si :

Une requête modifie les propriétés du contrôleur – ces changements affecteraient les requêtes suivantes.

Des initialisations sont faites dans `__construct()` pour chaque requête – le constructeur n'est appelé qu'une fois par processus, pas à chaque requête.
