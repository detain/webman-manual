# crontab 定時任務元件

## 說明

`workerman/crontab` 類似於 Linux 的 crontab，不同的是 `workerman/crontab` 支援秒級定時。

時間說明：

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[可省略，若沒有第 0 位，則最小時間粒度為分鐘]
```

## 專案網址

https://github.com/walkor/crontab
  
## 安裝
 
```php
composer require workerman/crontab
```
  
## 使用

**步驟一：建立進程檔 `app/process/Task.php`**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {
    
        // 每秒執行一次
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // 每 5 秒執行一次
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // 每分鐘執行一次
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // 每 5 分鐘執行一次
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // 每分鐘的第一秒執行
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // 每天 7 點 50 分執行，此處省略了秒位
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**步驟二：設定進程隨 webman 啟動**
  
開啟設定檔 `config/process.php`，新增以下設定：

```php
return [
    ....其他設定此處省略....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**步驟三：重新啟動 webman**

> 注意：定時任務不會立即執行，所有定時任務要等到下一分鐘才會開始計時執行。

## 說明
crontab 並非非同步執行。例如在一個 task 進程中設定了 A、B 兩個計時器，皆為每秒執行一次，若 A 任務耗時 10 秒，則 B 必須等 A 執行完才能執行，導致 B 執行延遲。
若業務對時間間隔較敏感，應將這些敏感定時任務放到獨立進程執行，以免受其他定時任務影響。範例 `config/process.php` 設定如下：

```php
return [
    ....其他設定此處省略....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
將對時間敏感的定時任務放在 `process/Task1.php` 中，其餘定時任務放在 `process/Task2.php` 中。

更多 `config/process.php` 設定說明，請參考 [自訂進程](../process.md)
