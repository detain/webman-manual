# webman是甚麼

webman是一款基於Workerman構建的高效能服務框架，整合了HTTP、WebSocket、TCP、UDP等多種模組。透過常駐記憶體、協程、連線池等先進技術，webman不僅突破了傳統PHP的效能瓶頸，還極大地擴展了其應用場景。

此外，webman還提供了強大的外掛機制，使開發者能夠快速整合和複用其他開發者開發的功能模組。無論是構建網站、開發HTTP介面、實現即時通訊、搭建物聯網系統，還是開發遊戲、TCP/UDP服務、Unix Socket服務等，webman都能輕鬆應對，展現出卓越的效能與靈活性。

# webman理念
**以最小核心提供最大的擴展性與最強的效能。**

webman僅提供最核心的功能（路由、中介軟體、session、自訂程序介面）。其餘功能全部複用composer生態，這意味著你可以在webman裡使用最熟悉的功能組件，例如在資料庫方面開發者可以選擇使用Laravel的[illuminate/database](./db/tutorial.md)，也可以是ThinkPHP的[ThinkORM](./db/thinkorm.md)，還可以是其他組件如`Medoo`。在webman裡整合他們是非常容易的事情。

# webman具有以下特點

1、高穩定性。webman基於workerman開發，workerman一直是業界bug極少的高穩定性socket框架。

2、超高效能。webman效能高於傳統php-fpm框架10-100倍左右，比go的gin echo等框架效能高一倍左右。

3、高複用。無需修改，可以複用現有composer生態。

4、高擴展性。支援自訂程序，可以做workerman能做的任何事情。

5、超級簡單易用，學習成本極低，代碼書寫與傳統框架沒有區別。

6、支援[二進制打包](./others/bin.md)，無需PHP環境即可直接運行。

7、使用最為寬鬆友善的MIT開源協議。

# 專案地址
GitHub: https://github.com/walkor/webman **不要吝嗇你的小星星哦**

碼雲: https://gitee.com/walkor/webman **不要吝嗇你的小星星哦**

# 第三方權威壓測數據

[![](../assets/img/benchmark1.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

帶資料庫查詢業務，webman單機吞吐量達到39萬QPS，比傳統php-fpm架構的laravel框架高出近80倍。

[![](../assets/img/benchmarks-go.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

帶資料庫查詢業務，webman比同類型go語言的web框架效能高一倍左右。

以上數據來自[techempower.com](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)
