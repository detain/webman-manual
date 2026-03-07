# crontab समय-सारणी घटक

## विवरण

`workerman/crontab` लिनक्स के crontab जैसा है, अंतर यह है कि सेकंड स्तर पर समय-सारणी का समर्थन करता है।

समय प्रारूप:

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[वैकल्पिक; न हो तो न्यूनतम इकाई मिनट है]
```

## परियोजना URL

https://github.com/walkor/crontab
  
## स्थापना
 
```php
composer require workerman/crontab
```
  
## उपयोग

**चरण 1: प्रक्रिया फ़ाइल `app/process/Task.php` बनाएं**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {

        // हर सेकंड चलाएं
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // हर 5 सेकंड चलाएं
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // हर मिनट चलाएं
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // हर 5 मिनट चलाएं
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // हर मिनट के पहले सेकंड में चलाएं
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // रोज़ 7:50 बजे चलाएं (यहाँ सेकंड छोड़ा गया है)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**चरण 2: webman के साथ प्रक्रिया शुरू करने का कॉन्फ़िगरेशन**
  
`config/process.php` खोलें और निम्न जोड़ें:

```php
return [
    ....अन्य कॉन्फ़िगरेशन छोड़ा गया....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**चरण 3: webman पुनः आरंभ करें**

> ध्यान दें: समय-सारणी कार्य तुरंत नहीं चलते; अगले मिनट से शुरू होते हैं।

## नोट
crontab असमकालिक नहीं है। जैसे: एक task प्रक्रिया में A और B दो टाइमर हैं, दोनों हर सेकंड चलते हैं। अगर कार्य A को 10 सेकंड लगते हैं तो B को A पूरा होने तक इंतज़ार करना पड़ता है, जिससे B की चलने में देरी होती है।
समय अंतराल के प्रति संवेदनशील होने पर, संवेदनशील कार्यों को अलग प्रक्रियाओं में चलाएं। उदाहरण `config/process.php`:

```php
return [
    ....अन्य कॉन्फ़िगरेशन छोड़ा गया....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
समय-संवेदनशील कार्य `process/Task1.php` में रखें, शेष `process/Task2.php` में।

`config/process.php` पर अधिक जानकारी के लिए [कस्टम प्रक्रिया](../process.md) देखें।
