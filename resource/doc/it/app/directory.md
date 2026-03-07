# Struttura della directory

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

Un plugin di applicazione ha la stessa struttura di directory e file di configurazione di webman. In pratica, l'esperienza di sviluppo è pressoché identica allo sviluppo di un'applicazione webman normale.

Le directory e la nomenclatura dei plugin seguono la specifica PSR-4. Poiché i plugin si trovano nella directory `plugin`, tutti i namespace iniziano con `plugin`, ad esempio `plugin\foo\app\controller\UserController`.

## Riguardo alla directory api

Ogni plugin ha una directory `api`. Se la tua applicazione fornisce interfacce interne da richiamare da altre applicazioni, inseriscile nella directory `api`.

Nota: le interfacce citate sono interfacce di chiamata a funzioni, non interfacce di rete/HTTP.

Ad esempio, il plugin email fornisce l'interfaccia `Email::send()` in `plugin/email/api/Email.php` per consentire ad altre applicazioni di inviare email. Inoltre, `plugin/email/api/Install.php` viene generato automaticamente per il mercato dei plugin webman-admin per eseguire operazioni di installazione o disinstallazione.
