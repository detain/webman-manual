# 目錄結構

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

應用插件擁有與 webman 相同的目錄結構與設定檔，實際開發體驗與開發一般 webman 應用程式幾乎沒有差異。

插件目錄與命名遵循 PSR-4 規範，由於插件皆放置於 `plugin` 目錄下，命名空間皆以 `plugin` 開頭，例如 `plugin\foo\app\controller\UserController`。

## 關於 api 目錄

每個插件都有 `api` 目錄。若你的應用需要提供供其他應用呼叫的內部介面，請將介面放在 `api` 目錄中。

注意：此處所稱介面為函式呼叫介面，而非網路／HTTP 介面。

例如，郵件插件在 `plugin/email/api/Email.php` 提供 `Email::send()` 介面，供其他應用發送郵件時呼叫。另外，`plugin/email/api/Install.php` 由系統自動產生，供 webman-admin 插件市場執行安裝或解除安裝操作。
