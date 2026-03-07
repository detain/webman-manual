# File di configurazione delle route
Il file di configurazione delle route dei plugin si trova in `plugin/nome_plugin/config/route.php`.

## Route predefinita
Tutti i percorsi URL dei plugin applicazione iniziano con `/app`, ad esempio l'URL di `plugin\foo\app\controller\UserController` è `http://127.0.0.1:8787/app/foo/user`.

## Disabilitare la route predefinita
Per disabilitare la route predefinita di un plugin applicazione, aggiungere quanto segue nella configurazione delle route:
```php
Route::disableDefaultRoute('foo');
```

## Gestire il callback 404
Per impostare un fallback per un plugin applicazione, è necessario passare il nome del plugin come secondo parametro. Esempio:
```php
Route::fallback(function(){
    return redirect('/');
}, 'foo');
```
