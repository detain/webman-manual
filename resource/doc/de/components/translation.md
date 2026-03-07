# Mehrsprachigkeit

Für die Mehrsprachigkeit wird die [symfony/translation](https://github.com/symfony/translation)-Komponente verwendet.

## Installation
```
composer require symfony/translation
```

## Sprachpakete erstellen
webman legt Sprachpakete standardmäßig im Verzeichnis `resource/translations` ab (bei Bedarf bitte anlegen). Änderungen an diesem Verzeichnis erfolgen in `config/translation.php`.
Jede Sprache entspricht einem Unterordner, die Sprachangebote liegen standardmäßig in `messages.php`. Beispiel:
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

Alle Sprachdateien geben ein Array zurück, z. B.:
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## Konfiguration

`config/translation.php`

```php
return [
    // Standardsprache
    'locale' => 'zh_CN',
    // Fallback-Sprache: Wenn eine Übersetzung in der aktuellen Sprache fehlt, wird auf die Fallback-Sprache zurückgegriffen
    'fallback_locale' => ['zh_CN', 'en'],
    // Ordner für Sprachdateien
    'path' => base_path() . '/resource/translations',
];
```

## Übersetzung

Zum Übersetzen wird die Methode `trans()` verwendet.

Sprachdatei `resource/translations/zh_CN/messages.php` anlegen:
```php
return [
    'hello' => '你好 世界!',
];
```

Datei `app/controller/UserController.php` anlegen:
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

Ein Aufruf von `http://127.0.0.1:8787/user/get` liefert „你好 世界!“.

## Standardsprache ändern

Zum Wechseln der Sprache wird die Methode `locale()` verwendet.

Sprachdatei `resource/translations/en/messages.php` hinzufügen:
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
        // Sprache wechseln
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
Ein Aufruf von `http://127.0.0.1:8787/user/get` liefert „hello world!“.

Die vierte Parameter der Funktion `trans()` kann für einen temporären Sprachwechsel verwendet werden. Das obige Beispiel entspricht z. B.:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // 4. Parameter wechselt die Sprache
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## Sprache pro Request explizit setzen
translation ist ein Singleton, d. h. alle Requests teilen sich dieselbe Instanz. Wenn ein Request per `locale()` die Standardsprache setzt, betrifft das alle nachfolgenden Requests im Prozess. Daher sollte die Sprache für jeden Request explizit gesetzt werden, z. B. mit folgendem Middleware:

Datei `app/middleware/Lang.php` anlegen (Verzeichnis ggf. vorher erstellen):
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

In `config/middleware.php` die globale Middleware hinzufügen:
```php
return [
    // Globale Middleware
    '' => [
        // ... weitere Middleware hier ausgelassen
        app\middleware\Lang::class,
    ]
];
```


## Platzhalter verwenden
Manchmal enthält eine Nachricht übersetzbare Variablen, z. B.
```php
trans('hello ' . $name);
```
In solchen Fällen werden Platzhalter verwendet.

`resource/translations/zh_CN/messages.php` entsprechend anpassen:
```php
return [
    'hello' => '你好 %name%!',
];
```
Beim Übersetzen werden die Werte über den zweiten Parameter übergeben:
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## Pluralisierung
In manchen Sprachen ändert sich der Satzbau je nach Anzahl. So ist „There is %count% apple“ nur bei `%count%` = 1 korrekt, bei größeren Werten nicht.

In solchen Fällen werden die Pluralformen mit einem **Pipe** (`|`) getrennt.

In der Sprachdatei `resource/translations/en/messages.php` `apple_count` ergänzen:
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

Auch Zahlbereiche sind möglich, um komplexere Pluralregeln zu definieren:
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## Sprachdatei festlegen

Die Standarddatei heißt `messages.php`. Es können jedoch auch andere Dateinamen verwendet werden.

Sprachdatei `resource/translations/zh_CN/admin.php` anlegen:
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

Über den dritten Parameter von `trans()` die Sprachdatei angeben (ohne `.php`-Erweiterung):
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## Weitere Informationen
Siehe [symfony/translation-Dokumentation](https://symfony.com/doc/current/translation.html)
