# Fichier de configuration des routes
Le fichier de configuration des routes des plugins se trouve dans `plugin/nom_du_plugin/config/route.php`.

## Route par défaut
Tous les chemins d'URL des plugins d'application commencent par `/app`. Par exemple, l'URL de `plugin\foo\app\controller\UserController` est `http://127.0.0.1:8787/app/foo/user`.

## Désactiver la route par défaut
Pour désactiver la route par défaut d'un plugin d'application, ajoutez ce qui suit dans la configuration des routes :
```php
Route::disableDefaultRoute('foo');
```

## Gérer le rappel 404
Pour définir un fallback pour un plugin d'application, il faut passer le nom du plugin en deuxième paramètre. Exemple :
```php
Route::fallback(function(){
    return redirect('/');
}, 'foo');
```
