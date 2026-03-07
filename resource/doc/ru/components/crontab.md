# Компонент периодических задач crontab

## Описание

`workerman/crontab` похож на Linux crontab, но поддерживает планирование с точностью до секунды.

Формат времени:

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[можно опустить; если нет 0-го разряда, минимальная единица — минута]
```

## URL проекта

https://github.com/walkor/crontab
  
## Установка
 
```php
composer require workerman/crontab
```
  
## Использование

**Шаг 1: Создать файл процесса `app/process/Task.php`**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {

        // Выполнять каждую секунду
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Выполнять каждые 5 секунд
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Выполнять каждую минуту
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Выполнять каждые 5 минут
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Выполнять в первую секунду каждой минуты
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // Выполнять в 7:50 каждый день (разряд секунд здесь опущен)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**Шаг 2: Настроить запуск процесса вместе с webman**
  
Открыть файл `config/process.php` и добавить:

```php
return [
    ....остальная конфигурация опущена....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**Шаг 3: Перезапустить webman**

> Примечание: Периодические задачи не запускаются сразу; счёт начинается со следующей минуты.

## Примечания
crontab не асинхронный. Пример: в процессе task настроены два таймера A и B, оба запускаются каждую секунду. Если задача A занимает 10 секунд, B ждёт завершения A — выполнение B задерживается.
Если логика чувствительна к интервалам, выносите чувствительные задачи в отдельные процессы, чтобы избежать влияния других. Пример для `config/process.php`:

```php
return [
    ....остальная конфигурация опущена....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
Чувствительные к времени задачи — в `process/Task1.php`, остальные — в `process/Task2.php`.

Подробнее о `config/process.php` см. [Пользовательские процессы](../process.md).
