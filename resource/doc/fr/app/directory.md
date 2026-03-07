# Structure du répertoire

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

Un plugin d’application possède la même structure de répertoires et les mêmes fichiers de configuration que webman. En pratique, l’expérience de développement est quasiment identique à celle d’une application webman classique.

Les répertoires et la nomenclature des plugins suivent la spécification PSR-4. Comme les plugins se trouvent dans le répertoire `plugin`, tous les espaces de noms commencent par `plugin`, par exemple `plugin\foo\app\controller\UserController`.

## À propos du répertoire api

Chaque plugin dispose d’un répertoire `api`. Si votre application fournit des interfaces internes appelées par d’autres applications, placez-les dans le répertoire `api`.

Remarque : les interfaces mentionnées ici sont des interfaces d’appel de fonction, et non des interfaces réseau/HTTP.

Par exemple, le plugin e-mail fournit l’interface `Email::send()` dans `plugin/email/api/Email.php` pour que d’autres applications puissent envoyer des e-mails. De plus, `plugin/email/api/Install.php` est généré automatiquement pour que le marché de plugins webman-admin exécute l’installation ou la désinstallation.
