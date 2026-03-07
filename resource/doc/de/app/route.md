# Routenkonfigurationsdatei
Die Routenkonfigurationsdatei von Plugins befindet sich unter `plugin/Pluginname/config/route.php`.

## Standardroute
Alle URL-Pfade von Anwendungs-Plugins beginnen mit `/app`, z. B. ist die URL für `plugin\foo\app\controller\UserController` `http://127.0.0.1:8787/app/foo/user`.

## Standardroute deaktivieren
Um die Standardroute eines Anwendungs-Plugins zu deaktivieren, fügen Sie Folgendes in der Routenkonfiguration hinzu:
```php
Route::disableDefaultRoute('foo');
```

## 404-Rückruf verarbeiten
Um einen Fallback für ein Anwendungs-Plugin festzulegen, müssen Sie den Plugin-Namen als zweiten Parameter übergeben. Beispiel:
```php
Route::fallback(function(){
    return redirect('/');
}, 'foo');
```
