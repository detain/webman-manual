# crontab 定期タスクコンポーネント

## 説明

`workerman/crontab` は Linux の crontab に似ていますが、秒単位のスケジューリングに対応しています。

時間の指定：

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[省略可。0の桁がない場合、最小単位は分]
```

## プロジェクト URL

https://github.com/walkor/crontab
  
## インストール
 
```php
composer require workerman/crontab
```
  
## 使用方法

**ステップ1: プロセスファイル `app/process/Task.php` を作成**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {
    
        // 1秒ごとに実行
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // 5秒ごとに実行
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // 1分ごとに実行
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // 5分ごとに実行
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // 毎分の1秒目に実行
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // 毎日7時50分に実行（秒は省略）
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**ステップ2: webman と同時にプロセスを起動するよう設定**
  
設定ファイル `config/process.php` を開き、以下を追加：

```php
return [
    ....その他の設定は省略....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**ステップ3: webman を再起動**

> 注意：定期タスクは即座には実行されません。次の分からカウントを開始し実行されます。

## 補足
crontab は非同期ではありません。例：1つの task プロセスに A と B の2つのタイマーを設定し、どちらも1秒ごとに実行する場合、タスク A に10秒かかると、B は A の完了を待つ必要があり、B の実行が遅れます。
時間間隔に敏感な処理の場合は、他のタスクの影響を受けないよう、時間に敏感な定期タスクを別プロセスで実行してください。例：`config/process.php` を次のように設定：

```php
return [
    ....その他の設定は省略....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
時間に敏感な定期タスクは `process/Task1.php` に、その他は `process/Task2.php` に配置してください。

`config/process.php` の詳細は [カスタムプロセス](../process.md) を参照してください。
