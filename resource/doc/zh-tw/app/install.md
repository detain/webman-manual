# 安裝

應用插件安裝有兩種方式：

## 在插件市場安裝
進入 [官方管理後台webman-admin](https://www.workerman.net/plugin/82) 的應用插件頁點擊安裝按鈕安裝對應的應用插件。

## 源碼包安裝
從應用市場下載應用插件壓縮包，解壓並將解壓目錄上傳到`{主項目}/plugin/`目錄下(如plugin目錄不存在需要手動創建)，執行 `php webman app-plugin:install 插件名`完成安裝。

例如下載的壓縮包名稱為ai.zip，解壓到 `{主項目}/plugin/ai`，執行`php webman app-plugin:install ai` 完成安裝。


# 卸載

同樣應用插件卸載也有兩種方式：

## 在插件市場卸載
進入 [官方管理後台webman-admin](https://www.workerman.net/plugin/82) 的應用插件頁點擊卸載按鈕卸載對應的應用插件。

## 源碼包卸載
執行 `php webman app-plugin:uninstall 插件名`完成卸載，執行完後手動刪除`{主項目}/plugin/`目錄下對應的插件目錄。
