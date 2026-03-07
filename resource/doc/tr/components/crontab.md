# crontab zamanlanmış görev bileşeni

## Açıklama

`workerman/crontab`, Linux crontab'ına benzer; fark olarak saniye düzeyinde zamanlama destekler.

Zaman formatı:

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[isteğe bağlı; yoksa minimum birim dakikadır]
```

## Proje URL'si

https://github.com/walkor/crontab
  
## Kurulum
 
```php
composer require workerman/crontab
```
  
## Kullanım

**Adım 1: `app/process/Task.php` işlem dosyasını oluşturun**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {

        // Her saniye çalıştır
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Her 5 saniyede çalıştır
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Her dakika çalıştır
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Her 5 dakikada çalıştır
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Her dakikanın ilk saniyesinde çalıştır
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // Her gün 7:50'de çalıştır (saniye burada atlandı)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**Adım 2: İşlemi webman ile birlikte başlatacak şekilde yapılandırın**
  
`config/process.php` dosyasını açıp aşağıdakini ekleyin:

```php
return [
    ....diğer yapılandırma atlandı....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**Adım 3: webman'i yeniden başlatın**

> Not: Zamanlanmış görevler hemen çalışmaz; sonraki dakikadan itibaren sayılmaya ve çalıştırılmaya başlar.

## Açıklama
crontab asenkron değildir. Örnek: Bir task işleminde A ve B iki zamanlayıcı var, ikisi de her saniye çalışıyor. A görevi 10 saniye sürerse B, A bitene kadar beklemek zorunda kalır, B'nin çalışması gecikir.
Zaman aralığına duyarlıysa, hassas zamanlanmış görevleri diğerlerinden etkilenmemesi için ayrı işlemde çalıştırın. Örnek `config/process.php` yapılandırması:

```php
return [
    ....diğer yapılandırma atlandı....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
Zamana duyarlı zamanlanmış görevleri `process/Task1.php` içine, diğerlerini `process/Task2.php` içine koyun.

`config/process.php` hakkında daha fazlası için [Özel İşlem](../process.md) bölümüne bakın.
