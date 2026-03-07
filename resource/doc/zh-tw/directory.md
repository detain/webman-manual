# 目錄結構
```
.
├── app                           應用目錄
│   ├── controller                控制器目錄
│   ├── model                     模型目錄
│   ├── view                      視圖目錄
│   ├── middleware                中介軟體目錄
│   │   └── StaticFile.php        內建靜態檔案中介軟體
│   ├── process                   自訂行程目錄
│   │   ├── Http.php              Http 行程
│   │   └── Monitor.php           監控行程
│   └── functions.php             業務自訂函數寫在這個檔案裡
├── config                        設定目錄
│   ├── app.php                   應用程式設定
│   ├── autoload.php              此處設定的檔案會被自動載入
│   ├── bootstrap.php             行程啟動時 onWorkerStart 執行的回呼設定
│   ├── container.php             容器設定
│   ├── dependence.php            容器依賴設定
│   ├── database.php              資料庫設定
│   ├── exception.php             例外設定
│   ├── log.php                   日誌設定
│   ├── middleware.php            中介軟體設定
│   ├── process.php               自訂行程設定
│   ├── redis.php                 Redis 設定
│   ├── route.php                 路由設定
│   ├── server.php                埠號、行程數等伺服器設定
│   ├── view.php                  視圖設定
│   ├── static.php                靜態檔案開關及靜態檔案中介軟體設定
│   ├── translation.php           多語言設定
│   └── session.php               Session 設定
├── public                        靜態資源目錄
├── runtime                       應用程式執行時目錄，需具寫入權限
├── start.php                     服務啟動檔案
├── vendor                        composer 安裝的第三方類別庫目錄
└── support                       類別庫適配（含第三方類別庫）
    ├── Request.php               請求類別
    ├── Response.php              回應類別
    ├── Setup.php                 安裝導向腳本
    └── bootstrap.php             行程啟動後初始化腳本
```
