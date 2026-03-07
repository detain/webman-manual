# Структура каталогов

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

Плагин приложения имеет ту же структуру каталогов и файлов конфигурации, что и webman. На практике опыт разработки практически не отличается от разработки обычного приложения webman.

Каталоги и именование плагинов соответствуют спецификации PSR-4. Поскольку все плагины размещаются в каталоге `plugin`, пространства имён начинаются с `plugin`, например `plugin\foo\app\controller\UserController`.

## О каталоге api

У каждого плагина есть каталог `api`. Если ваше приложение предоставляет внутренние интерфейсы для вызова из других приложений, поместите их в каталог `api`.

Примечание: здесь речь идёт об интерфейсах вызова функций, а не о сетевых/HTTP-интерфейсах.

Например, плагин электронной почты предоставляет интерфейс `Email::send()` в `plugin/email/api/Email.php` для вызова другими приложениями при отправке писем. Кроме того, `plugin/email/api/Install.php` генерируется автоматически для выполнения установки и удаления через магазин плагинов webman-admin.
