# À propos des fuites de mémoire
Webman est un framework résident en mémoire, il faut donc porter une certaine attention aux fuites de mémoire. Les développeurs n’ont toutefois pas lieu de s’inquiéter outre mesure, car les fuites surviennent uniquement dans des conditions très extrêmes et sont faciles à éviter. L’expérience de développement avec webman est essentiellement la même qu’avec les frameworks traditionnels ; aucune opération supplémentaire n’est nécessaire pour la gestion de la mémoire.

> **Remarque**
> Le processus moniteur intégré de webman surveille l’utilisation mémoire de tous les processus. Lorsque l’utilisation mémoire d’un processus est sur le point d’atteindre la valeur définie dans `memory_limit` de php.ini, le processus correspondant sera redémarré automatiquement et de manière sécurisée pour libérer la mémoire, sans impact sur l’application.

## Définition d’une fuite de mémoire
Que l’utilisation mémoire de webman augmente avec le nombre de requêtes est normal. En général, une fois qu’un processus atteint un certain volume de requêtes (typiquement de l’ordre du million), la mémoire se stabilise ou n’augmente plus que légèrement et occasionnellement.

Pour la plupart des applications, la mémoire utilisée par un processus finit par se stabiliser autour de 10 M–100 M. Pas de quoi s’inquiéter tant que la mémoire par processus reste sous 100 M.

De plus, lors du traitement de gros fichiers, de grosses requêtes ou de lectures importantes depuis la base de données, PHP alloue une quantité importante de mémoire. PHP peut conserver une partie de cette mémoire pour la réutiliser plutôt que de la restituer au système d’exploitation, ce qui peut entraîner une utilisation élevée. Comme cette mémoire est réutilisée, il n’y a pas lieu de s’inquiéter.

> **Remarque**
> Pour les projets empaquetés en phar ou en binaire, si la taille du paquet est importante, une utilisation mémoire supérieure à 100 M est normale.

## Comment confirmer une fuite de mémoire
Si un processus a traité plus d’un million de requêtes, que l’utilisation mémoire dépasse 100 M et que la mémoire continue à augmenter après chaque requête, une fuite de mémoire est possible.

## Comment localiser une fuite de mémoire
Une approche simple consiste à soumettre chaque API à des tests de charge et à identifier celle dont l’utilisation mémoire continue à augmenter après des millions de requêtes.

Une fois l’API concernée identifiée, on peut procéder par dichotomie : commenter à chaque fois la moitié du code métier jusqu’à isoler le code en cause.

## Comment se produisent les fuites de mémoire
**Une fuite de mémoire ne se produit que lorsque les deux conditions suivantes sont remplies :**
1. Il existe un tableau à **cycle de vie long** (les tableaux ordinaires ne posent pas de problème)
2. Et ce tableau à **cycle de vie long** grandit indéfiniment (l’application continue à y insérer des données sans jamais les nettoyer)

Une fuite ne survient que si **les deux** conditions sont remplies. Si l’une manque ou qu’une seule est remplie, il ne s’agit pas d’une fuite.

## Tableaux à cycle de vie long

Dans webman, les tableaux à cycle de vie long incluent :
1. Les tableaux avec le mot-clé `static`
2. Les propriétés de type tableau des singletons
3. Les tableaux avec le mot-clé `global`

> **Remarque**
> Les données à cycle de vie long sont autorisées dans webman, mais il faut s’assurer que les données restent bornées et que le nombre d’éléments ne grandit pas indéfiniment.

Voici des exemples pour chaque cas.

### Tableau static à expansion infinie
```php
class Foo
{
    public static $data = [];
    public function index(Request $request)
    {
        self::$data[] = time();
        return response('hello');
    }
}
```

Le tableau `$data` défini avec `static` a un cycle de vie long. Dans l’exemple, `$data` grandit à chaque requête, ce qui provoque une fuite de mémoire.

### Propriété de tableau de singleton à expansion infinie
```php
class Cache
{
    protected static $instance;
    public $data = [];
    
    public function instance()
    {
        if (!self::$instance) {
            self::$instance = new self;
        }
        return self::$instance;
    }
    
    public function set($key, $value)
    {
        $this->data[$key] = $value;
    }
}
```

Code d’appel
```php
class Foo
{
    public function index(Request $request)
    {
        Cache::instance()->set(time(), time());
        return response('hello');
    }
}
```

`Cache::instance()` renvoie un singleton Cache à cycle de vie long. Même si la propriété `$data` n’utilise pas `static`, la classe ayant un cycle de vie long, `$data` est aussi un tableau à cycle de vie long. À mesure que des clés différentes sont ajoutées à `$data`, l’utilisation mémoire augmente et une fuite se produit.

> **Remarque**
> Si les clés ajoutées par `Cache::instance()->set(key, value)` sont en nombre limité, il n’y a pas de fuite, car le tableau `$data` ne grandit pas indéfiniment.

### Tableau global à expansion infinie
```php
class Index
{
    public function index(Request $request)
    {
        global $data;
        $data[] = time();
        return response($foo->sayHello());
    }
}
```
Les tableaux définis avec `global` ne sont pas libérés à la fin de la fonction ou de la méthode ; ils ont donc un cycle de vie long. Le code ci-dessus provoque une fuite à mesure que les requêtes augmentent. De même, les tableaux définis avec `static` à l’intérieur d’une fonction ou d’une méthode sont aussi des tableaux à cycle de vie long ; s’ils grandissent indéfiniment, une fuite se produit, par exemple :
```php
class Index
{
    public function index(Request $request)
    {
        static $data = [];
        $data[] = time();
        return response($foo->sayHello());
    }
}
```

## Recommandations
Il est conseillé de ne pas trop se concentrer sur les fuites de mémoire, car elles sont rares. Si une fuite survient, des tests de charge permettent de retrouver le code responsable. Même si le développeur ne trouve pas la cause, le service moniteur intégré à webman redémarrera à temps le processus concerné pour libérer la mémoire.

Si vous souhaitez réduire les risques de fuite, vous pouvez suivre ces recommandations :
1. Éviter autant que possible les tableaux avec `global` ou `static` ; si vous les utilisez, assurez-vous qu’ils ne grandissent pas indéfiniment.
2. Pour les classes peu connues, privilégier l’initialisation avec `new` plutôt que des singletons. Si un singleton est nécessaire, vérifier s’il a des propriétés de tableau susceptibles de grandir indéfiniment.
