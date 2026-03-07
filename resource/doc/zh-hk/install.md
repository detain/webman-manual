# 如何安裝 webman

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux 安裝 PHP + Composer 環境（已有環境請略過）
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **注意**
> 以上指令適用於 Linux/Mac 系統，Windows 系統需自行安裝 PHP 環境。

亦可手動下載 webman 官方提供的[靜態 PHP](https://www.workerman.net/download)，解壓後即可使用。

## 1. 建立項目

```php
composer create-project workerman/webman:~2.0
```

> **提示**
> 若出現錯誤，可能是使用了有問題的 Composer 鏡像代理，請執行 `composer config -g --unset repos.packagist` 取消代理。

## 2. 執行

進入 webman 目錄

#### Windows 用戶
雙擊 `windows.bat` 或執行 `php windows.php` 啟動

> **提示**
> 若有錯誤，可能是函數被禁用，請參考[函數禁用檢查](others/disable-function-check.md)解除禁用。

#### Linux 用戶
**除錯模式**（用於開發除錯：輸出會顯示於終端，終端關閉後 webman 服務也會關閉）

```php
php start.php start
```

**守護進程模式**（用於正式環境：輸出不會顯示於終端，終端關閉後 webman 服務會持續執行）

```php
php start.php start -d
```

#### Docker 用戶

啟動所有服務並附加至主控台
```php
docker-compose up
```

在背景模式執行服務
```php
docker-compose up -d
```

> **提示**
> 若有錯誤，可能是函數被禁用，請參考[函數禁用檢查](others/disable-function-check.md)解除禁用。

## 3. 存取

瀏覽器開啟 `http://IP位址:8787`
