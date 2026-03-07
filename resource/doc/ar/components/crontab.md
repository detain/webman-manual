# مكوّن المهام المجدولة crontab

## الوصف

`workerman/crontab` شبيه بـ crontab في Linux، والفرق أنه يدعم الجدولة حتى مستوى الثانية.

تنسيق الوقت:

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[اختياري؛ إذا غاب، أدنى وحدة وقت هي الدقيقة]
```

## رابط المشروع

https://github.com/walkor/crontab
  
## التثبيت
 
```php
composer require workerman/crontab
```
  
## الاستخدام

**الخطوة 1: إنشاء ملف العملية `app/process/Task.php`**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {

        // التشغيل كل ثانية
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // التشغيل كل 5 ثوانٍ
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // التشغيل كل دقيقة
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // التشغيل كل 5 دقائق
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // التشغيل في الثانية الأولى من كل دقيقة
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // التشغيل يومياً الساعة 7:50 (حقل الثواني مُحذوف هنا)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**الخطوة 2: إعداد العملية للتشغيل مع webman**
  
افتح ملف الإعداد `config/process.php` وأضف التالي:

```php
return [
    ....إعدادات أخرى محذوفة....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**الخطوة 3: إعادة تشغيل webman**

> ملاحظة: المهام المجدولة لا تُنفَّذ فوراً؛ تبدأ العد والتنفيذ من الدقيقة التالية.

## ملاحظات
crontab غير غير متزامن. مثلاً: في عملية task هناك مؤقتان A و B، كلاهما يعمل كل ثانية. إن استغرقت المهمة A 10 ثوانٍ، فإن B يجب أن تنتظر انتهاء A قبل التنفيذ، فيتأخر تنفيذ B.
إذا كانت منطقك حساساً لفترات الوقت، شغّل المهام الحساسة للوقت في عمليات منفصلة لتجنب التداخل. مثال لـ `config/process.php`:

```php
return [
    ....إعدادات أخرى محذوفة....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
ضع المهام الحساسة للوقت في `process/Task1.php` والباقي في `process/Task2.php`.

للمزيد عن `config/process.php` راجع [العمليات المخصصة](../process.md).
