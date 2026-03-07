# 執行流程

## 進程啟動流程

執行 php start.php start 後執行流程如下：

1. 載入 config/ 下的配置
2. 設置 Worker 的相關配置如 `pid_file` `stdout_file` `log_file` `max_package_size` 等
3. 創建 webman 進程，並監聽端口（默認 8787）
4. 根據配置創建自定義進程
5. webman 進程和自定義進程啟動後執行以下邏輯（以下都是執行在 onWorkerStart 裡）：
   ① 載入 `config/autoload.php` 裡設置的檔案，如 `app/functions.php`
   ② 載入 `config/middleware.php`（包括 `config/plugin/*/*/middleware.php`）裡設置的中間件
   ③ 執行 `config/bootstrap.php`（包括 `config/plugin/*/*/bootstrap.php`）裡設置類的 start 方法，用於初始化一些模組，比如 Laravel 資料庫初始化連接
   ④ 載入 `config/route.php`（包括 `config/plugin/*/*/route.php`）裡定義的路由

## 處理請求流程

1. 判斷請求 url 是否對應 public 下的靜態檔案，是的話返回檔案（結束請求），不是的話進入 2
2. 根據 url 判斷是否命中某個路由，沒命中進入 3、命中進入 4
3. 是否關閉了默認路由，是的話返回 404（結束請求），不是的話進入 4
4. 找到請求對應控制器的中間件，按順序執行中間件前置操作（洋蔥模型請求階段），執行控制器業務邏輯，執行中間件後置操作（洋蔥模型響應階段），請求結束。（參考[中間件洋蔥模型](https://www.workerman.net/doc/webman/middleware.html#%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%B4%8B%E8%91%B1%E6%A8%A1%E5%9E%8B))
