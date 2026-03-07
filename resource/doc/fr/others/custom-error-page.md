# Personnaliser 404

Si vous souhaitez contrôler dynamiquement le contenu du 404, par exemple renvoyer des données JSON `{"code:"404", "msg":"404 not found"}` pour les requêtes AJAX et renvoyer le modèle `app/view/404.html` pour les requêtes de page, consultez l'exemple suivant.

> L'exemple ci-dessous utilise des modèles PHP natifs. Les autres modèles comme `twig`, `blade`, `think-template` suivent le même principe.

**Créer le fichier `app/view/404.html`**
```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>404 not found</title>
</head>
<body>
<?=htmlspecialchars($error)?>
</body>
</html>
```

**Ajouter le code suivant dans `config/route.php` :**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // Renvoyer du JSON pour les requêtes AJAX
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // Renvoyer le modèle 404.html pour les requêtes de page
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# Personnaliser 405

Depuis webman-framework 1.5.23, le callback de repli accepte un paramètre `status`. 404 signifie que la requête n'existe pas ; 405 signifie que la méthode de requête actuelle n'est pas autorisée (par ex. accéder en GET à une route définie avec `Route::post()`).

```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request, $status) {
    $map = [
        404 => '404 not found',
        405 => '405 method not allowed',
    ];
    return response($map[$status], $status);
});
```

# Personnaliser 500

**Créer `app/view/500.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
Modèle d'erreur personnalisé :
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**Créer `app/exception/Handler.php`** (créer le répertoire s'il n'existe pas)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * Rendu et renvoi de la réponse
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // Renvoyer des données JSON pour les requêtes AJAX
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // Renvoyer le modèle 500.html pour les requêtes de page
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**Configurer `config/exception.php`**
```php
return [
    '' => \app\exception\Handler::class,
];
```
