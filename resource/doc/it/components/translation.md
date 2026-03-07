# Multilingua

La funzionalità multilingua utilizza il componente [symfony/translation](https://github.com/symfony/translation).

## Installazione
```
composer require symfony/translation
```

## Creare i pacchetti di lingua
webman memorizza i pacchetti di lingua di default nella cartella `resource/translations` (crearla se non esiste). Per cambiare la cartella, configurare in `config/translation.php`.
Ogni lingua corrisponde a una sottocartella, le definizioni sono memorizzate in `messages.php` di default. Esempio:
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

Tutti i file di lingua restituiscono un array, ad esempio:
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## Configurazione

`config/translation.php`

```php
return [
    // Lingua predefinita
    'locale' => 'zh_CN',
    // Lingua di fallback: se la traduzione non viene trovata nella lingua corrente, si prova con la lingua di fallback
    'fallback_locale' => ['zh_CN', 'en'],
    // Cartella dove sono memorizzati i file di lingua
    'path' => base_path() . '/resource/translations',
];
```

## Traduzione

Per tradurre si usa il metodo `trans()`.

Creare il file di lingua `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 世界!',
];
```

Creare il file `app/controller/UserController.php`:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        $hello = trans('hello'); // 你好 世界!
        return response($hello);
    }
}
```

Accedendo a `http://127.0.0.1:8787/user/get` verrà restituito "你好 世界!"

## Cambiare la lingua predefinita

Usare il metodo `locale()` per cambiare lingua.

Aggiungere il file di lingua `resource/translations/en/messages.php`:
```php
return [
    'hello' => 'hello world!',
];
```

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // Cambiare lingua
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
Accedendo a `http://127.0.0.1:8787/user/get` verrà restituito "hello world!"

Si può anche usare il 4° parametro della funzione `trans()` per cambiare temporaneamente lingua. L'esempio sopra è equivalente a:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // Il 4° parametro cambia la lingua
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## Impostare la lingua esplicitamente per ogni richiesta
translation è un singleton, quindi tutte le richieste condividono la stessa istanza. Se una richiesta imposta la lingua predefinita con `locale()`, influenzerà tutte le richieste successive del processo. Bisogna quindi impostare la lingua esplicitamente per ogni richiesta, ad esempio con il seguente middleware:

Creare il file `app/middleware/Lang.php` (creare la cartella se non esiste):
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Lang implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        locale(session('lang', 'zh_CN'));
        return $handler($request);
    }
}
```

Aggiungere il middleware globale in `config/middleware.php`:
```php
return [
    // Middleware globale
    '' => [
        // ... altri middleware omessi
        app\middleware\Lang::class,
    ]
];
```


## Usare i segnaposto
A volte un messaggio contiene variabili da tradurre, ad esempio
```php
trans('hello ' . $name);
```
In questi casi si usano i segnaposto.

Modificare `resource/translations/zh_CN/messages.php`:
```php
return [
    'hello' => '你好 %name%!',
];
```
Durante la traduzione, i valori si passano tramite il 2° parametro:
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## Gestire i plurali
In alcune lingue la struttura della frase cambia in base alla quantità. Ad esempio, "There is %count% apple" è corretto quando `%count%` è 1, ma errato quando è maggiore.

In questi casi si usa il **pipe** (`|`) per elencare le forme plurali.

Aggiungere `apple_count` nel file `resource/translations/en/messages.php`:
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

Si possono anche specificare intervalli numerici per regole di plurale più complesse:
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## Specificare il file di lingua

Il nome predefinito è `messages.php`, ma si possono creare file con altri nomi.

Creare il file `resource/translations/zh_CN/admin.php`:
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

Specificare il file con il 3° parametro di `trans()` (omettere l'estensione `.php`):
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## Maggiori informazioni
Vedere la [documentazione symfony/translation](https://symfony.com/doc/current/translation.html)
