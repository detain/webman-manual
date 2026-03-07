# 自動加載

## 利用 Composer 加載 PSR-0 規範的檔案
webman 遵循 `PSR-4` 自動加載規範。若你的業務需要加載 `PSR-0` 規範的程式庫，請參考以下操作。

- 新建 `extend` 目錄用於存放 `PSR-0` 規範的程式庫
- 編輯 `composer.json`，在 `autoload` 下加入以下內容：

```json
"psr-0" : {
    "": "extend/"
}
```
最終結果類似：
![](../../assets/img/psr0.png)

- 執行 `composer dumpautoload`
- 執行 `php start.php restart` 重啟 webman（注意：必須重啟才能生效）

## 利用 Composer 加載某些檔案

- 編輯 `composer.json`，在 `autoload.files` 下加入要加載的檔案：
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- 執行 `composer dumpautoload`
- 執行 `php start.php restart` 重啟 webman（注意：必須重啟才能生效）

> **提示**
> `composer.json` 中 `autoload.files` 配置的檔案會在 webman 啟動前加載；而透過框架 `config/autoload.php` 加載的檔案則在 webman 啟動後才加載。
> `composer.json` 中 `autoload.files` 加載的檔案變更後必須 restart 才會生效，reload 不生效。透過框架 `config/autoload.php` 加載的檔案支援熱加載，變更後 reload 即可生效。

## 利用框架加載某些檔案
有些檔案可能不符合 PSR 規範，無法自動加載，可透過配置 `config/autoload.php` 加載這些檔案，例如：
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **提示**
 > `autoload.php` 中設置了加載 `support/Request.php` 和 `support/Response.php` 兩個檔案，因為 `vendor/workerman/webman-framework/src/support/` 下也有同名的檔案。透過 `autoload.php` 優先加載專案根目錄下的這兩個檔案，可讓你自訂其內容而無需修改 `vendor` 中的檔案。若不需要自訂，可略過這兩項配置。

