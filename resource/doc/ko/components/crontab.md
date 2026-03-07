# crontab 스케줄 컴포넌트

## 설명

`workerman/crontab`은 Linux crontab과 유사하지만, 초 단위 스케줄을 지원합니다.

시간 형식:

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[생략 가능, 0번째가 없으면 최소 단위는 분]
```

## 프로젝트 주소

https://github.com/walkor/crontab
  
## 설치
 
```php
composer require workerman/crontab
```
  
## 사용법

**1단계: 프로세스 파일 `app/process/Task.php` 생성**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {
    
        // 매 초 실행
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // 5초마다 실행
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // 매 분 실행
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // 5분마다 실행
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // 매 분 1초에 실행
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // 매일 7시 50분 실행 (초는 생략)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**2단계: webman과 함께 프로세스 시작되도록 설정**
  
설정 파일 `config/process.php`를 열고 다음을 추가:

```php
return [
    ....기타 설정 생략....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**3단계: webman 재시작**

> 참고: 스케줄 작업은 즉시 실행되지 않으며, 다음 분부터 카운트하여 실행됩니다.

## 참고
crontab은 비동기가 아닙니다. 예: 한 task 프로세스에 A, B 두 타이머를 설정하고 둘 다 1초마다 실행하는데, A 작업이 10초 걸리면 B는 A가 끝날 때까지 대기해야 하므로 B 실행이 지연됩니다.
시간 간격에 민감한 경우, 다른 작업에 영향받지 않도록 민감한 스케줄 작업을 별도 프로세스에서 실행하세요. 예: `config/process.php`에서 다음과 같이 설정:

```php
return [
    ....기타 설정 생략....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
시간에 민감한 스케줄 작업은 `process/Task1.php`에, 그 외는 `process/Task2.php`에 두세요.

`config/process.php` 자세한 내용은 [커스텀 프로세스](../process.md)를 참조하세요.
