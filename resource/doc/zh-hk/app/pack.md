# 打包

例如打包foo應用插件

* 設置`plugin/foo/config/app.php`裡版本號(**重要**)
* 刪除`plugin/foo`裡不需要打包的文件，尤其是`plugin/foo/public`下測試上傳功能的臨時文件
* 如果你的項目包含數據庫建表等操作，需要設置好`plugin/foo/install.sql`，參見[安裝數據庫部分](database.md#自動導入資料庫)
* 如果你的項目有自己獨立的數據庫、Redis配置，需要先刪除這些配置，這些配置應該是在首次訪問應用時觸發安裝引導程序(需要自行實現)，讓管理員手動填寫並生成。
* 如果你的項目包含webman admin後台菜單，需要設置好 `plugin/foo/config/menu.php`，這樣安裝插件時會自動設置這些菜單。具體參見[webman-admin導入菜單](https://www.workerman.net/doc/webman-admin/app-development/menu.html)
* 恢復其它需要恢復原貌的文件
* 完成以上操作後進入`{主項目}/plugin/`目錄
* Linux用戶使用命令 `zip -r foo.zip foo` 生成foo.zip
* Windows用戶右鍵foo文件夾選擇`壓縮為ZIP文件` 生成foo.zip

**foo.zip為打包後的文件，參考下一章節[發佈插件](publish.md)**
