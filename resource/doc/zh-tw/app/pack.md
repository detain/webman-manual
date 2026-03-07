# 打包

例如打包foo應用程式插件

* 在`plugin/foo/config/app.php`設定版本號(**重要**)
* 刪除`plugin/foo`中不需打包的檔案，特別是`plugin/foo/public`下用於測試上傳功能的臨時檔案
* 若您的專案包含資料庫建表等操作，需正確設定`plugin/foo/install.sql`，參見[安裝資料庫部分](database.md)
* 若您的專案有獨立之資料庫、Redis 設定，需先移除這些設定。這些設定應於首次存取應用程式時，透過安裝精靈觸發（需自行實作），讓管理員手動填寫並產生。
* 若您的專案包含 webman admin 後台選單，需正確設定 `plugin/foo/config/menu.php`，安裝插件時會自動設定這些選單。詳見 [webman-admin 匯入選單](https://www.workerman.net/doc/webman-admin/app-development/menu.html)
* 還原其他須恢復原貌的檔案
* 完成以上操作後進入`{主專案}/plugin/`目錄
* Linux 用戶：使用指令 `zip -r foo.zip foo` 產生 foo.zip
* Windows 用戶：於 foo 資料夾上按右鍵，選擇「壓縮為 ZIP 檔案」產生 foo.zip

**foo.zip 為打包後的檔案，請參考下一章節 [發佈插件](publish.md)**
