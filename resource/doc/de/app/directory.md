# Verzeichnisstruktur

```
plugin/
└── foo
    ├── app
    │   ├── controller
    │   │   └── IndexController.php
    │   ├── exception
    │   │   └── Handler.php
    │   ├── functions.php
    │   ├── middleware
    │   ├── model
    │   └── view
    │       └── index
    │           └── index.html
    ├── config
    │   ├── app.php
    │   ├── autoload.php
    │   ├── container.php
    │   ├── database.php
    │   ├── exception.php
    │   ├── log.php
    │   ├── middleware.php
    │   ├── process.php
    │   ├── redis.php
    │   ├── route.php
    │   ├── static.php
    │   ├── thinkorm.php
    │   ├── translation.php
    │   └── view.php
    ├── public
    └── api
```

Ein Anwendungs-Plugin besitzt dieselbe Verzeichnisstruktur und Konfigurationsdateien wie webman. In der Praxis ist die Entwicklungserfahrung praktisch identisch mit der Entwicklung einer normalen webman-Anwendung.

Plugin-Verzeichnisse und -Benennung folgen der PSR-4-Spezifikation. Da alle Plugins im Verzeichnis `plugin` liegen, beginnen alle Namensräume mit `plugin`, z. B. `plugin\foo\app\controller\UserController`.

## Über das api-Verzeichnis

Jedes Plugin hat ein `api`-Verzeichnis. Wenn Ihre Anwendung interne Schnittstellen bereitstellt, die von anderen Anwendungen aufgerufen werden, legen Sie diese Schnittstellen im `api`-Verzeichnis ab.

Hinweis: Die hier genannten Schnittstellen sind Funktionsaufruf-Schnittstellen, nicht Netzwerk-/HTTP-Schnittstellen.

Beispielsweise bietet das E-Mail-Plugin in `plugin/email/api/Email.php` die Schnittstelle `Email::send()` für andere Anwendungen zum Versenden von E-Mails. Außerdem wird `plugin/email/api/Install.php` automatisch erzeugt, damit der webman-admin-Plugin-Markt Installations- oder Deinstallationsvorgänge ausführen kann.
