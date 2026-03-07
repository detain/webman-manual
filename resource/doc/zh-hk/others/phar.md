# phar打包

phar是PHP裡類似於JAR的一種打包文件，你可以利用phar將你的webman項目打包成單個phar文件，方便部署。

**這裡非常感謝 [fuzqing](https://github.com/fuzqing) 的PR。**

> **注意**
> 需要關閉`php.ini`的phar配置選項，即設置 `phar.readonly = 0`

## 安裝命令行工具
`composer require webman/console`

## 打包
在webman項目根目錄執行命令 `php webman build:phar`
會在 build 目錄生成一個`webman.phar`文件。

> 打包相關配置在 `config/plugin/webman/console/app.php` 中

## 啟動停止相關命令
**啟動**
`php webman.phar start` 或 `php webman.phar start -d`

**停止**
`php webman.phar stop`

**查看狀態**
`php webman.phar status`

**查看連接狀態**
`php webman.phar connections`

**重啟**
`php webman.phar restart` 或 `php webman.phar restart -d`

## 說明
* 打包後的項目不支持reload，更新代碼需要restart重啟

* 為了避免打包文件尺寸過大佔用過多內存，可以設置 `config/plugin/webman/console/app.php`裡的`exclude_pattern` `exclude_files`選項將排除不必要的文件。

* 運行webman.phar後會在webman.phar所在目錄生成runtime目錄，用於存放日誌等臨時文件。

* 如果你的項目裡使用了.env文件，需要將.env文件放在webman.phar所在目錄。

* 切勿將用戶上傳的文件存儲在phar包中，因為以`phar://`協議操作用戶上傳的文件是非常危險的(phar反序列化漏洞)。用戶上傳的文件必須單獨存儲在phar包之外的磁盤中，參見下面。

* 如果你的業務需要上傳文件到public目錄，需要將public目錄獨立出來放在webman.phar所在目錄，這時候需要配置`config/app.php`。
```
'public_path' => base_path(false) . DIRECTORY_SEPARATOR . 'public',
```
業務可以使用助手函數`public_path($文件相對位置)`找到實際的public目錄位置。

* 注意webman.phar不支持在Windows下開啟自定義進程
